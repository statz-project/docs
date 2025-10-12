# Descriptive Statistics in Stat-z

This tutorial shows how to use the **Stat-z core functions** to compute descriptive statistics for **categorical** and **numeric** variables.

---

## 1. Summarizing categorical variables

The function `summarize_q` computes absolute and relative frequencies.

### Example
```js
const values = ["Male", "Male", "Female", "Female", "Female"];

const result = window.Statz.summarize_q(values);

console.log(result);
```

### Output (simplified)

```json
{
  "columns": ["Variable", "Description"],
  "rows": [
    { "Variable": "Female", "Description": "3 (60.0%)" },
    { "Variable": "Male",   "Description": "2 (40.0%)" }
  ],
  "summary": { "total": 5 }
}
```

---

## 2. Summarizing list-type variables

If a variable contains **multiple items separated by a delimiter** (e.g., hobbies, symptoms), use `summarize_l`.

### Example

```js
const values = [
  "Headache;Nausea",
  "Headache",
  "Nausea;Fever",
  ""
];

const result = window.Statz.summarize_l(values, ";");

console.log(result);
```

### Output (simplified)

```json
{
  "columns": ["Variable", "Description"],
  "rows": [
    { "Variable": "Headache", "Description": "2 (50.0%)" },
    { "Variable": "Nausea",   "Description": "2 (50.0%)" },
    { "Variable": "Fever",    "Description": "1 (25.0%)" },
    { "Variable": "Not reported", "Description": "1 (25.0%)" }
  ]
}
```

---

## 3. Summarizing numeric variables

For numeric data, use `summarize_n` to compute mean, SD, median, IQR, etc.

### Example

```js
const values = ["5", "7", "6", "6", "10"];

const result = window.Statz.summarize_n(values);

console.log(result);
```

### Output (simplified)

```json
{
  "columns": ["Variable", "Description"],
  "rows": [
    { "Variable": "Mean (SD)", "Description": "6.8 (1.9)" },
    { "Variable": "Median (IQR)", "Description": "6.0 (2.0)" },
    { "Variable": "n", "Description": "5" }
  ],
  "summary": { "n": 5 }
}
```

---

## 4. Quick column previews

`describeColumn(column, variantIndex, options)` builds a short list of strings that summarises a column or one of its variants. It is ideal for Bubble repeating groups or tooltips where you need a compact snapshot of the data.

### Example
```js
const column = window.Statz.makeColumn([
  'Male',
  'Female',
  'Female',
  ''
], { col_hash: 'col_gender' });

const preview = window.Statz.describeColumn(column);

console.log(preview);
// ["Female: 2 (50.0%)", "Male: 1 (25.0%)", "Not reported: 1 (25.0%)"]
```

### Options
- `variantIndex`: pass an index from `col_vars` to preview a derived column instead of the base variant.
- `options.lang`: localise the summary using any supported Stat-z locale.
- `options.maxRows`: limit the number of lines returned for long factor lists.
- `options.formatFn`: provide a custom formatter used by the underlying summariser if you need bespoke strings.

---

## Tips

* Use `options.lang` to localize output (e.g., `"pt_br"`, `"en_us"`, `"es_es"`).
* Use `options.stat_options` to customize numeric summaries (mean, median, min/max, etc.); add `"n_missing"` to include a missing-value count (the row appears only when any value is missing).
* Results can be exported to **HTML**, **Markdown**, or later combined into full tables.

---

- Next tutorial: [Chi-square and Fisher's Exact Test](chi_fisher.md)
