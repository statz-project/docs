# Column Processing

## Overview

Processing options are stored in `column.meta.processing` and applied non-destructively at analysis time. Original data (`col_values`) is never modified by processing -- transformations are applied on-the-fly when `applyProcessing()` is called.

## Available Options

| Field | Type | Default | Applies to | Description |
|---|---|---|---|---|
| `sort_mode` | string | `"default"` | q | Level ordering: `"default"`, `"freq_desc"`, `"freq_asc"`, `"alpha"`, `"custom"` |
| `custom_order` | string[] | `[]` | q | Custom level order (when `sort_mode = "custom"`) |
| `top_n` | number\|null | `null` | q | Keep only top N levels; group rest under `top_n_label` |
| `top_n_label` | string | `"Others"` | q | Label for the grouped "others" category |
| `na_action` | string | `"keep"` | q, n, l | `"keep"` = leave as missing; `"label"` = replace ALL missing (originally empty + excluded) with `na_label` |
| `na_label` | string | `"Not informed"` | q, n, l | Replacement label when `na_action = "label"` |
| `excluded_values` | string[] | `[]` | q, n, l | Values to treat as missing. With `na_action: "keep"` they disappear; with `na_action: "label"` they collapse into `na_label`. |

## When Processing is Applied

- `runAnalysis`: applies processing before computing statistics
- `describeDataset` / `describeColumn`: applies processing before generating summaries
- `exportDatabaseAsHTML`: applies processing before rendering (opt-out via `{ applyProcessing: false }`)
- `buildMissingMap` / `exportMissingMapAsHTML`: apply processing before counting missing values (opt-out via `{ applyProcessing: false }`)
- `buildCrosstab` / `exportCrosstabAsHTML`: apply processing on **both** axes before cross-tabulating (opt-out via `{ applyProcessing: false }`)

## When Processing is NOT Applied

- `getIndividualItems`: returns raw values (for editing UI)
- `getIndividualItemsWithCount`: returns raw values (for editing UI)
- `combineAnalysisAsSingleTable`: operates on analysis results
- `exportCombinedAsHTML`: operates on analysis results

## Order of Application

0. **Replacements** (`meta.replacements`) -- applied lazily by `applyReplacements` (see [dataset-helpers.md](dataset-helpers.md#search--replace-values-in-a-column))
1. **Excluded values** (`excluded_values`) -- excluded entries become empty (treated as missing)
2. **NA handling** (`na_action`, `na_label`) -- when `na_action = "label"`, **all** empty rows (originally missing OR just excluded in step 1) are relabeled with `na_label`
3. **Sort levels** (`sort_mode`, `custom_order`) -- qualitative only; the NA label participates in frequency ordering
4. **Top N grouping** (`top_n`, `top_n_label`) -- qualitative only; "Others" is the final result

Conceptually, `excluded_values` defines **what counts as missing** and `na_action`/`na_label` decides **how missing values are displayed**. The two are complementary: excluding a value with `na_action: "keep"` makes it disappear from the analysis; excluding with `na_action: "label"` collapses it into the NA label, alongside originally-empty rows.

The canonical read-time helper that runs the full pipeline (replacements + processing) is `factors.resolveColumn(column, options)`. The analysis pipeline (`runAnalysis`, `describeColumn`, `exportDatabaseAsHTML`) calls it automatically.

### How processing shows up in the missing-data map

`buildMissingMap` / `exportMissingMapAsHTML` ([export.md](export.md#8-missing-data-map)) count
missing values on the resolved view, so the two processing options that define missingness are
directly visible there:

| Processing | Effect on the map |
|---|---|
| `excluded_values` | Those rows become empty in step 1 → they **appear as holes**. |
| `na_action: "label"` | Step 2 turns every empty row into a real category → the column reports **zero** missing. |

Both are intended: the map answers "how much data is missing **after** my edits", the same question
`runAnalysis` answers. `{ applyProcessing: false }` is the escape hatch for seeing the original
import. The canonical per-value test is `factors.isMissingValue(value, col_type, col_sep)`.

## Usage

```javascript
// Full pipeline (replacements + processing) — preferred
const resolved = Statz.resolveColumn(column);

// Just the processing step (skip replacements)
const processed = Statz.applyProcessing(column);
const processed2 = Statz.applyProcessing(column, { excluded_values: ['unknown'] });

// Automatic in analysis (no extra code needed)
// runAnalysis reads meta.processing from each column automatically
const result = Statz.runAnalysis(predictors, responses, databases, options);
```

## Data Structure

```json
{
  "col_name": "sex",
  "col_values": { "col_compact": true, "labels": ["male", "female", "unknown"], "codes": [1, 2, 1, 3] },
  "meta": {
    "replacements": [],
    "processing": {
      "sort_mode": "freq_desc",
      "top_n": 2,
      "top_n_label": "Others",
      "excluded_values": ["unknown"],
      "na_action": "label",
      "na_label": "Not informed"
    }
  }
}
```
