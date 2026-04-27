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
| `na_action` | string | `"keep"` | q, n, l | `"keep"` = leave as missing; `"label"` = replace with `na_label` |
| `na_label` | string | `"Not informed"` | q, n, l | Replacement label when `na_action = "label"` |
| `excluded_values` | string[] | `[]` | q, n, l | Values to exclude from analysis |

## When Processing is Applied

- `runAnalysis`: applies processing before computing statistics
- `describeDataset` / `describeColumn`: applies processing before generating summaries
- `exportDatabaseAsHTML`: applies processing before rendering (opt-out via `{ applyProcessing: false }`)

## When Processing is NOT Applied

- `getIndividualItems`: returns raw values (for editing UI)
- `getIndividualItemsWithCount`: returns raw values (for editing UI)
- `combineAnalysisAsSingleTable`: operates on analysis results
- `exportCombinedAsHTML`: operates on analysis results

## Order of Application

0. **Replacements** (`meta.replacements`) -- already baked into `col_values` by `replaceColumnValues` (destructive, before `applyProcessing` runs)
1. **Excluded values** (`excluded_values`) -- excluded entries are set to empty
2. **NA handling** (`na_action`, `na_label`) -- only labels rows that were *originally* missing; rows that became empty due to step 1 stay empty
3. **Sort levels** (`sort_mode`, `custom_order`) -- qualitative only; the NA label participates in frequency ordering
4. **Top N grouping** (`top_n`, `top_n_label`) -- qualitative only; "Others" is the final result

Each step operates on progressively cleaner data: exclusions are removed first so NA / sort / top_n act only on kept values, while still distinguishing user-excluded entries from genuine missing data.

## Usage

```javascript
// Direct usage
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
