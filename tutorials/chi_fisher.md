# Chi-square and Fisher’s Exact Test

This tutorial explains how to perform **association tests between two categorical variables** in Stat-z, using the functions:

- `window.Statz.summarize_q_q` → generates contingency tables with percentages.  
- `window.Statz.computePValueQxQ` → calculates the p-value (Chi-square or Fisher, depending on the case).  

---

## 1. Basic 2x2 table (small sample)

Let’s test whether **Gender** is associated with **Response**:

```js
const predictor = ["Male", "Male", "Female", "Female"];
const response  = ["Yes", "No", "Yes", "No"];

const result = window.Statz.summarize_q_q(predictor, response);

console.log(result);
````

### Output (simplified)

```json
{
  "columns": ["Group", "No", "Yes", "p-value"],
  "rows": [
    { "Group": "Female", "No": "1 (50.0%)", "Yes": "1 (50.0%)", "p-value": "" },
    { "Group": "Male",   "No": "1 (50.0%)", "Yes": "1 (50.0%)", "p-value": "" }
  ],
  "test_used": "Fisher’s Exact Test",
  "p_value": 1.0000
}
```

✅ Because expected frequencies are small, Statz automatically applied **Fisher’s Exact Test**.

---

## 2. Larger contingency table (>2x2)

Now let’s test **Education Level** versus **Response**:

```js
const predictor = [
  "High School","High School","Bachelor","Bachelor","Master","PhD"
];
const response  = [
  "Yes","No","Yes","No","Yes","No"
];

const result = window.Statz.summarize_q_q(predictor, response);

console.log(result);
```

### Output (simplified)

```json
{
  "columns": ["Group", "No", "Yes", "p-value"],
  "rows": [
    { "Group": "Bachelor",   "No": "1 (50.0%)", "Yes": "1 (50.0%)", "p-value": "" },
    { "Group": "High School","No": "1 (50.0%)", "Yes": "1 (50.0%)", "p-value": "" },
    { "Group": "Master",     "No": "0 (0.0%)",  "Yes": "1 (100.0%)","p-value": "" },
    { "Group": "PhD",        "No": "1 (100.0%)","Yes": "0 (0.0%)",  "p-value": "" }
  ],
  "test_used": "Chi-square",
  "p_value": 0.7135
}
```

For tables larger than 2x2, Stat-z uses the **Chi-square test**.

* If the global test is significant, adjusted residuals will be computed and annotated with symbols (†, \*).

---

## 3. Forcing Chi-square in 2x2

By default, Stat-z uses **Fisher** when expected counts < 5 in a 2x2.
If you want to **force Chi-square with Yates’ correction**, pass an option:

```js
const observed = [
  [2, 1],
  [1, 2]
];

const result = window.Statz.computePValueQxQ(observed, { correct: true });

console.log(result);
```

### Output (simplified)

```json
{
  "method": "Chi-square",
  "p_value": 1.0000
}
```

## Summarizing list predictors vs qualitative responses

Sometimes a predictor column is a **list field** (e.g., `"fever;headache"`).  

Use `window.Statz.summarize_l_q` to automatically split the list into binary indicators and run `summarize_q_q` for each item.

```js
const clinics = [
  "headache;overweight",
  "dm;fever",
  "headache",
  ""
];

const income = ["low", "high", "high", "low"];

const results = window.Statz.summarize_l_q(
  clinics,
  income,
  null,
  { lang: "en_us" },
  { predictorLabel: "Clinics" }
);

results.forEach(entry => {
  console.log(entry.label, entry.table.test_used, entry.table.p_value);
});
```

Each entry in `results` represents a 2×2 chi-square (or Fisher) test comparing the presence/absence of a list value (e.g., `"headache"`) versus the qualitative response (`income` in the example).

---

## Interpretation tips

* **Fisher’s Exact Test** → best for small 2x2 tables (low expected frequencies).
* **Chi-square Test** → default for larger tables.
* **Residuals (†, \*)** → mark cells with frequencies higher/lower than expected when the association is significant.

---

## Next steps

* Try [Non-parametric tests](nonparametric.md) for numeric variables vs. groups.

---
