# Creating column variants

The core helper in `core/json/variants.js` lets you build derived column definitions without mutating the original data. A variant is simply another entry in a column's `col_vars` array, carrying its own encoded values plus metadata that describes how it was produced. This guide walks through the pipeline that powers `variants.createVariant` and shows how to compose the most common transformations.

For end-to-end examples of creating columns, reading data, and persisting variants, see [Working with dataset helpers](./dataset-helpers.md).

## How the helper works

`createVariant(baseColumn, config)` first **resolves the source** (applies the source's `meta.replacements` and `meta.processing` via `factors.resolveColumn`) so the variant operates on the user's effective view of the data, not raw `col_values`. Then it decodes those resolved values, applies a sequence of steps, and returns a new `ColumnVariant` object. The steps run in this order:

1. `fillEmpty` -> replace blank entries with the provided fallback.
2. `replacements` / `searchReplace` -> rewrite individual values (works with list-style entries as well).
3. `merges` -> collapse multiple discrete values into named buckets.
4. `subsetLevels` -> keep only the levels you care about.
5. `forceNumeric` -> coerce cleaned text into numeric strings, logging drops and edits.
6. `transform` -> apply `log`, `log10`, `log2`, `sqrt`, or `square` to numeric inputs.
7. `cut` -> convert numeric values into labeled intervals.
8. Explicit overrides (`col_type`, `col_sep`, `sortByFrequency`, `note`).

Because source resolution runs first, configurations like `excluded_values` or `replacements` set on the source column propagate naturally into the variant: a sentinel marked `excluded_values: ["999"]` will arrive at the pipeline as empty, not as `999`. The variant's own operations target the resolved values (e.g., if the source has `replacements: [{from:'m', to:'male'}]`, your variant's `replacements: [{from:'male', to:'Male'}]` matches the resolved label, not the raw `m`).

At every stage the helper records `meta.actions` and `meta.warnings`, so the UI can display what happened. When you append the returned variant to an existing column, the original data remains untouched.

## Quick start

```js
import variants from '../core/json/variants.js';
import factors from '../core/json/factors.js';

const baseCol = factors.makeColumn([
  'Yes',
  'No',
  '',
  'Prefer not to say',
  'Yes'
], { encode: false });

const cleanedVariant = variants.createVariant(baseCol, {
  kind: 'cleaned',
  var_label: 'Q1 (cleaned)',
  fillEmpty: 'No response',
  replacements: [
    { search: 'Prefer not to say', replace: 'No response' }
  ],
  sortByFrequency: true,
  note: 'Stacks blanks and opt-outs together.'
});

baseCol.col_vars.push(cleanedVariant);
```

`cleanedVariant.meta.actions` captures the operations (`fill_missing`, `search_replace`, `sort_by_frequency`), and `meta.warnings` lists any observed changes -- ready to surface in logs or tooltips.

## Common recipes

### Clean up categorical responses

Use string helpers together to standardize free-form survey answers:

```js
const locale = 'pt-BR';
const categoricalVariant = variants.createVariant(baseCol, {
  kind: 'standardized',
  replacements: [
    { search: 'N/A', replace: '' },
    { search: 'Prefer not to say', replace: '' }
  ],
  merges: [
    { label: 'Positive', levels: ['Very satisfied', 'Satisfied'] },
    { label: 'Negative', levels: ['Dissatisfied', 'Very dissatisfied'] }
  ],
  subsetLevels: ['Positive', 'Negative'],
  fillEmpty: 'No response',
  sortByFrequency: true,
  lang: locale
});
```

List-type columns (`col_type === 'l'`) are handled automatically: the helper splits on the active separator, applies replacements or merges to each item, and rejoins the list.

### Transform numeric values

Coerce messy numeric text and apply a transform in one pass:

```js
const incomeVariant = variants.createVariant(baseCol, {
  kind: 'numeric',
  forceNumeric: {},
  transform: { fn: 'log10' },
  var_label: 'Household income (log10)',
  note: 'Skips non-positive entries; see warnings for dropped rows.'
});

if (incomeVariant.meta.warnings.length) {
  console.warn(incomeVariant.meta.warnings.join('\n'));
}
```

The helper strips currency symbols, normalizes commas vs. dots, and reports replacements or discarded rows via warnings.

### Bucket a numeric scale

`cut` splits numeric values into labeled intervals. You can feed explicit breakpoints or let the helper derive them from a fixed width:

```js
const buckets = variants.createVariant(baseCol, {
  kind: 'binned',
  forceNumeric: {},
  cut: {
    width: 10,
    includeLowest: true,
    labels: ['0-10', '10-20', '20-30', '30+']
  },
  var_label: 'Score buckets'
});

console.table(buckets.meta.breaks);
```

`meta.breaks` stores the `[lower, upper]` pairs that define each interval, and `meta.labels` mirrors the labels that were applied. Entries outside the range raise a warning and are returned as blank strings.

## Configuration reference

**Source selection**
- `sourceVarIndex` chooses an existing variant (`baseCol.col_vars[index]`) as the starting point. Defaults to the original column values.
- `kind`, `var_label`, and `label` control how the variant is presented.
- `lang` influences translated warning messages.

**Categorical cleanup**
- `fillEmpty` replaces empty strings before any other operation.
- `replacements` / `searchReplace` accept `{ search, replace }` pairs; helpers also understand Bubble-style `{ from, to }` or `{ level, label }` keys.
- `merges` collapse multiple levels into a single label (`{ label, levels: [] }`).
- `subsetLevels` whitelists values to keep; everything else becomes blank.
- `sortByFrequency` reorders factor labels based on observed frequency and remaps the encoded codes to match.

**Numeric handling**
- `forceNumeric` coerces each value to a canonical numeric string. Non-parsable entries turn into empty strings.
- `transform` applies math functions to the coerced values (`log`, `log10`, `log2`, `sqrt`, `square`). Invalid inputs are skipped and reported.
- `cut` buckets numeric values into intervals. Provide either `breaks` (explicit array) or `width` (equal-width bins). Optional keys: `origin`, `right`, `includeLowest`, `labels`.

**Output overrides**
- `col_type` and `col_sep` force the encoding of the resulting variant.
- `note` attaches free-form context that consumers can display alongside the variant.

## Understanding the metadata

Every variant returns:
- `meta.actions` -> ordered array describing each transformation (type plus contextual payload).
- `meta.warnings` -> translated strings summarizing noteworthy edits or dropped records.
- `meta.breaks` / `meta.labels` -> present when `cut` runs, so visual components can render the interval table.
- `meta.source_type` and `meta.source_var_index` -> useful when showing "derived from" badges in the UI.

You can surface the metadata anywhere you list variants -- for example, in audit logs, tooltips, or download manifests.

## Variant templates

`variants.VARIANT_TEMPLATES` exposes presets for qualitative (`'q'`), numeric (`'n'`), and list (`'l'`) columns. Each template lists the configurable options needed to power quick actions in the UI. Start with these IDs when wiring dropdowns or buttons in Bubble, and pass the matching options into `createVariant`.

Each template carries both a static English `label` (backward-compat fallback) and a `labelKey` that resolves through the `core/i18n` module. For localized UI labels, prefer `variants.getVariantTemplates(lang)` — it returns a deep-cloned copy of `VARIANT_TEMPLATES` with each `label` resolved for the requested language (`'en_us'`, `'pt_br'`, `'es_es'`):

```js
const templates = window.Statz.getVariantTemplates('pt_br').q;
// templates[0].label === 'Buscar e substituir níveis'
// templates[0].labelKey === 'variants.templates.search_replace.q'
```

Translation keys live under `variants.templates.*` in `core/i18n/index.js`. The static `VARIANT_TEMPLATES.[type][i].label` field remains for code that does not need localization.

`variants.TRANSFORM_ORDER` gives the canonical pipeline order of the operations. Use it to render steps in sequence and to reset downstream panels when an upstream step is edited.

`variants.OPERATION_DEFAULTS` maps each recipe operation key to a sensible initial value, so the UI can seed a freshly-enabled operation in the recipe draft:

```js
// When the user enables a template, initialize its draft state:
const optionKey = template.options[0];                 // e.g. 'cut'
const draft = structuredClone(window.Statz.OPERATION_DEFAULTS[optionKey]);
// draft === { breaks: [], labels: [], right: true, includeLowest: true }
```

The keys align with `VARIANT_TEMPLATES[type][].options[]` and the canonical recipe shape produced by `normalizeRecipe`. Note the search & replace default is keyed `replacements` (not `searchReplace`). Always deep-clone the default before mutating it — `OPERATION_DEFAULTS` is a shared constant.

With these building blocks you can script reproducible data-cleaning steps, keep survey scripts tidy, and help Bubble users understand exactly how each derived column was produced.
