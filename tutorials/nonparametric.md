
# Non-parametric Tests

This tutorial explains how to use Stat-z to perform **non-parametric tests**, which are useful when the assumption of normality or homoscedasticity does not hold.  

Covered here:
- **Mann–Whitney (Wilcoxon Rank-Sum)** → two groups.  
- **Kruskal–Wallis** → more than two groups.  
- **Dunn’s post-hoc test** → multiple comparisons after Kruskal–Wallis.  

---

## 1. Mann–Whitney Test (two groups)

The function `summarize_n_q` automatically chooses the correct test depending on normality and group count.  

### Example
```js
const predictor = ["5","7","6","6","10","12"];   // numeric
const response  = ["Group A","Group A","Group A","Group B","Group B","Group B"]; // categorical

const result = window.Utils.summarize_n_q(predictor, response);

console.log(result);
````

### Output (simplified)

```json
{
  "columns": ["Group", "Group A", "Group B", "p-value"],
  "rows": [
    { "Group": "Mean (SD)", "Group A": "6.0 (0.8)", "Group B": "11.0 (1.0)", "p-value": "" },
    { "Group": "Median (IQR)", "Group A": "6.0 (1.0)", "Group B": "11.0 (1.0)", "p-value": "" }
  ],
  "test_used": "Mann-Whitney",
  "p_value": 0.0500
}
```

✅ Stat-z automatically used **Mann–Whitney** because groups are not normally distributed.

---

## 2. Kruskal–Wallis Test (more than two groups)

### Example

```js
const predictor = ["5","7","6","6","10","12","8","9","15"];
const response  = ["Group A","Group A","Group A","Group B","Group B","Group B","Group C","Group C","Group C"];

const result = window.Utils.summarize_n_q(predictor, response);

console.log(result);
```

### Output (simplified)

```json
{
  "columns": ["Group", "Group A", "Group B", "Group C", "p-value"],
  "rows": [
    { "Group": "Mean (SD)", "Group A": "6.0 (0.8)", "Group B": "11.0 (1.0)", "Group C": "10.7 (3.5)", "p-value": "" },
    { "Group": "Median (IQR)", "Group A": "6.0 (1.0)", "Group B": "11.0 (1.0)", "Group C": "9.0 (4.0)", "p-value": "" }
  ],
  "test_used": "Kruskal-Wallis",
  "p_value": 0.0410,
  "posthoc": [
    { "groupA": "Group A", "groupB": "Group B", "pValue": 0.0200, "significant": true },
    { "groupA": "Group A", "groupB": "Group C", "pValue": 0.1800, "significant": false },
    { "groupA": "Group B", "groupB": "Group C", "pValue": 0.2500, "significant": false }
  ]
}
```

The overall **Kruskal–Wallis test** is significant (`p = 0.0410`).
Stat-z automatically runs **Dunn’s post-hoc test** with Bonferroni correction.

---

## 3. Dunn’s Post-hoc Test

When Kruskal–Wallis is significant, Stat-z outputs pairwise comparisons.

Example above shows:

* **Group A vs Group B**: significant difference (`p = 0.0200`).
* Other comparisons: not significant.

You can also export significant post-hoc results with:

```js
const html = window.Utils.exportPosthocComparisonsAsHTML([result]);

console.log(html);
```

This produces an HTML table summarizing significant comparisons.

---

## Interpretation tips

* **Mann–Whitney** is the non-parametric alternative to the t-test (two groups).
* **Kruskal–Wallis** is the non-parametric alternative to one-way ANOVA (≥3 groups).
* **Dunn’s test** is required to identify *which groups differ* when Kruskal–Wallis is significant.

---

## Next steps

* Try [ANOVA and Tukey’s HSD](anova_tukey.md) *(future tutorial)*.
* Explore [Chi-square vs Fisher](chi_fisher.md) for categorical predictors.

