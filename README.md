# Stat-z Documentation

Welcome to the **documentation hub** of Stat-z.  
Here you will find tutorials, references, and usage guides for both the **open-source analytical core** and the **Stat-z SaaS platform**.

---

## What is Stat-z?

Stat-z is a **hybrid project** that combines:
- An **open-source core in JavaScript**: statistical utilities and JSON data handling.  
- A **SaaS interface built with Bubble**: a no-code platform designed to make statistical analysis accessible to non-experts.  
- Optional **R APIs** for advanced analyses (logistic regressions, mixed models, survival).  

Our mission:  
*Democratize statistics for students, researchers, and professionals without advanced training.*

---

## Repository structure

- [`core`](https://github.com/stat-z/core) → JavaScript analytical utilities (open-source).  
- [`examples`](https://github.com/stat-z/examples) → Example scripts and datasets.  
- [`docs`](https://github.com/stat-z/docs) → Documentation (this repository).  
- *(future)* `r-api` → R wrappers for advanced statistics. 

---

## Quick Start

1. Clone the `core` repository or include the script in your project:
```bash
   git clone https://github.com/stat-z/core.git
````

2. Use the statistical utilities in your JS code:

```js
   const predictor = ["Male","Male","Female","Female"];
   const response  = ["Yes","No","Yes","No"];

   const result = window.Statz.summarize_q_q(predictor, response);

   console.log(result);
```

3. See [`examples`](https://github.com/stat-z/examples) for datasets and ready-to-use demos.

---

## Tutorials

* [Importing worksheet data locally](tutorials/import.md)
* [Descriptive statistics (categorical & numeric)](tutorials/descriptive.md)
* [Chi-square and Fisher’s exact test](tutorials/chi_fisher.md)
* [Non-parametric tests (Mann–Whitney, Kruskal–Wallis, Dunn)](tutorials/nonparametric.md)
* [Creating column variants](tutorials/variant.md)
* [Exporting results (HTML, Markdown)](tutorials/export.md)

*(more tutorials coming soon)*

---

## Roadmap (Documentation)

* [ ] Write beginner-friendly tutorial: *“Running your first analysis in 5 minutes”*.
* [ ] Add API reference (auto-generated from `core`).
* [ ] Expand tutorials with academic use cases.

---

## Contributing

Improvements to the documentation are welcome:

1. Check the open [Issues](https://github.com/statz-project/docs/issues).
2. Fork this repository.
3. Add or improve a tutorial.
4. Submit a Pull Request.

For contributing to the JavaScript core (code, tests, features), see the Core guide:

- https://github.com/statz-project/core/blob/main/CONTRIBUTING.md

---

## License

The documentation is open-source under the **CC-BY 4.0 License**.
The analytical core is licensed under **AGPL v3**.

---

Together, we can make statistics **accessible to everyone**.


