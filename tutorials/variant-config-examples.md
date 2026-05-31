# Variant config examples

Minimal JSON examples of the `config` object accepted by `createVariant(baseCol, config)`. One block per operation. Use these as the shape the Bubble UI must produce per panel.

## Common shell

Every config carries a small set of meta fields (none required by the pipeline, but recommended for traceability):

```json
{
  "var_label": "Sex (cleaned)",
  "kind": "search_replace",
  "sourceVarIndex": 0,
  "note": "Optional free-text note shown in tooltips/audit",
  "lang": "pt_br"
}
```

- `var_label` — display label of the variant.
- `kind` — semantic category; usually matches the template id (e.g., `"search_replace"`, `"cut_intervals"`).
- `sourceVarIndex` — index in `col_vars` of the source variant; `0` = base/original; omit or set to `null` for the same.
- `note` — optional user note.
- `lang` — locale for translated warnings (`"en_us"`, `"pt_br"`, `"es_es"`).

The blocks below add one operation at a time on top of this shell.

---

## 1. `fillEmpty` — fill missing cells

```json
{
  "var_label": "Income (filled)",
  "kind": "fill_missing",
  "sourceVarIndex": 0,
  "fillEmpty": "Not informed"
}
```

Scalar value (string or number). Replaces every empty/whitespace cell with the given value.

---

## 2. `replacements` — search & replace levels

```json
{
  "var_label": "Sex (PT)",
  "kind": "search_replace",
  "sourceVarIndex": 0,
  "replacements": [
    { "from": "male", "to": "Homem" },
    { "from": "female", "to": "Mulher" }
  ]
}
```

Canonical key is `replacements`; the helper also accepts `searchReplace` and per-entry aliases (`search`/`from`/`value`/`level` and `replace`/`to`/`label`), all collapsed to `{from, to}` in the stored recipe. Entries with empty `from` are dropped. Empty `to` removes the matching items from list-type columns.

---

## 3. `merges` — group levels under a target label

```json
{
  "var_label": "Symptoms (grouped)",
  "kind": "merge_levels",
  "sourceVarIndex": 0,
  "merges": [
    { "label": "Respiratory", "levels": ["cough", "wheezing", "dyspnea"] },
    { "label": "Cardiac",     "levels": ["chest_pain", "palpitations"] }
  ]
}
```

Canonical keys are `label` and `levels`; the helper also accepts `target`/`name` for label and `values` for levels.

---

## 4. `subsetLevels` — keep only listed levels

```json
{
  "var_label": "Sex (binary only)",
  "kind": "subset",
  "sourceVarIndex": 0,
  "subsetLevels": ["male", "female"]
}
```

Cells not in the list become empty (missing).

---

## 5. `forceNumeric` — coerce to numeric

```json
{
  "var_label": "Age (numeric)",
  "kind": "coerce_numeric",
  "sourceVarIndex": 0,
  "forceNumeric": true
}
```

Any truthy value triggers coercion. Output `col_type` becomes `"n"`. Unparseable rows become empty and surface as warnings.

---

## 6. `transform` — numeric transform

```json
{
  "var_label": "Income (log10)",
  "kind": "transform",
  "sourceVarIndex": 0,
  "transform": { "fn": "log10" }
}
```

`fn` accepts `"log"`, `"log10"`, `"log2"`, `"sqrt"`, `"square"`. Output `col_type` becomes `"n"`. There is an option set "Transform type" that holds these functions (data source for the dropdown).

`base` is only meaningful when `fn: "log"` (defaults to `Math.E`):

```json
{
  "transform": { "fn": "log", "base": 2 }
}
```

---

## 7. `cut` — bin numeric values into intervals

There is an option set "Cut type" that holds the available options: width/explicit.

Mode A — explicit breaks:

```json
{
  "var_label": "Age bins",
  "kind": "cut_intervals",
  "sourceVarIndex": 0,
  "cut": {
    "breaks": [0, 18, 60, 120],
    "right": true,
    "includeLowest": true
  }
}
```

Mode B — equal-width bins:

```json
{
  "cut": {
    "width": 10,
    "origin": 0,
    "right": true,
    "includeLowest": true
  }
}
```

Optional `labels` to override generated interval labels (length must match number of intervals):

```json
{
  "cut": {
    "breaks": [0, 18, 60, 120],
    "labels": ["Child", "Adult", "Senior"]
  }
}
```

Defaults: `right: true`, `includeLowest: true` — both omitted from the stored recipe when left at default. Output `col_type` becomes `"q"`, with labels ordered by interval (natural order, not alphabetical).

---

## 8. `sortByFrequency` — reorder factor levels by frequency

```json
{
  "var_label": "Symptoms (freq order)",
  "kind": "sort_frequency",
  "sourceVarIndex": 0,
  "sortByFrequency": true
}
```

Reorders `col_values.labels` by descending observed frequency (alphabetical tiebreak) and remaps `codes` accordingly. Applies to `"q"` and `"l"` only.

---

## Final-type overrides

Two optional fields force the final shape regardless of the pipeline's inference:

```json
{
  "col_type": "l",
  "col_sep": ";"
}
```

Rarely needed. The pipeline already promotes/demotes the type when applicable (`forceNumeric`/`transform` → `"n"`; `cut` → `"q"`). Use only when you need an explicit final shape (e.g., converting a `"q"` result back to a single-item list).
