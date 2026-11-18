# Clinical prediction and survival analysis on a simulated Alzheimer's cohort

An end-to-end, heavily annotated notebook that walks from a relational clinical database
to prediction models, model explanation, and time-to-event analysis with competing risks.

Written from the perspective of a physician learning computational methods, so every
step states not only *how* it was done but *why*, and what its limitations are.

> [!IMPORTANT]
> **All data in this repository is synthetic.** The cohort is generated inside the
> notebook (section 2) from a fixed random seed. No real patient data is used at any
> point, and no result here is a clinical finding. The purpose is to demonstrate and
> validate an analysis pipeline where the ground truth is known by construction.

## Why this notebook exists

Most clinical ML tutorials stop at `fit`, `predict`, and an AUC. The questions that
decide whether an analysis is defensible come after that, and they rarely get taught:

- Does the imputation leak the test set into training?
- Is AUC the right metric when the outcome affects one patient in five?
- Does feature importance mean what the library implies it means?
- What happens to a ten-year risk estimate when patients die of something else first?

Because the data-generating process is known, every answer can be checked against the
truth. The notebook ends by asking the same question four ways and getting four
different numbers.

## What the notebook covers

**Part 1. Data and prediction (sections 1 to 17)**

| Section | Topic |
| --- | --- |
| 2 to 3 | Building a 4-table SQLite database and loading it with SQL |
| 4 to 5 | Data quality: implausible values, long-format missingness, long to wide |
| 6 to 9 | Table 1, MMSE trajectories, risk factors, correlation structure |
| 10 to 11 | Outcome definition, exclusions, leakage check, stratified split |
| 12 to 15 | Logistic regression, XGBoost, LightGBM, CatBoost, each in a pipeline |
| 16 | Final test-set evaluation: ROC, precision-recall, calibration, bootstrap CIs |
| 17 | SHAP: global beeswarm and an individual waterfall |

**Part 2. Survival analysis (sections 18 to 22)**

| Section | Topic |
| --- | --- |
| 18 | Building the time-to-event dataset from all 2,000 patients |
| 19 | Cumulative incidence with death as a competing risk |
| 20 | Cause-specific Cox model for Alzheimer's |
| 21 | Testing the proportional hazards assumption |
| 22 | Cox model for the competing event, with a forest plot |

The notebook closes with four **exercises**, each one testing a claim the notebook
itself makes. No solutions are provided.

## The result worth looking at first

The ten-year risk of Alzheimer's in the same simulated cohort, by method:

| Method | Estimate | Bias |
| --- | --- | --- |
| Simple proportion (243 / 2,000) | 12.2% | Underestimates: ignores incomplete follow-up |
| Logistic cohort prevalence (243 / 1,141) | 21.3% | Overestimates: survivor-only controls |
| Naive 1 − Kaplan-Meier | 17.2% | Overestimates: treats death as censoring |
| **Cumulative incidence, competing risks** | **14.7%** | **Correct** |

The four prediction models land within noise of each other (AUC 0.803 to 0.813, with
overlapping bootstrap intervals). Logistic regression has the best average precision
and Brier score, which is consistent with the additive logistic structure the data were
simulated from. That result is a feature of the exercise, not a claim about real data.

## Running it

**In the browser, no installation:**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/acpnavarro/alzheimer-cohort-ml-tutorial/blob/main/alzheimer_cohort.ipynb)

**Locally:**

```bash
git clone https://github.com/acpnavarro/alzheimer-cohort-ml-tutorial.git
cd alzheimer-cohort-ml-tutorial

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
jupyter lab                      # or open the folder in VS Code
```

Then run the notebook from the top. Section 2 writes `alzheimer_cohort.db` into the
working directory, so nothing needs to be downloaded. The seed is fixed, so the database
is byte-identical on every run and the numbers in this README reproduce exactly.

The database file is deliberately **not** committed. It is derived data that the
notebook regenerates in about twenty seconds, and committing it would contradict the
reproducibility the notebook argues for.

## Requirements

Python 3.14. Exact tested versions are in `requirements.txt`.

Note that this notebook was produced on **pandas 3.x**, which introduced breaking
changes relative to pandas 2.x. If you relax the pins, expect to adjust a few calls.

## Limitations

Stated at the end of each section and collected in the final summary. The main ones:

- Simulated data throughout. The pipeline is validated; the biology is not.
- The prediction cohort excludes 43% of patients whose ten-year status is unknown,
  so the reported performance is optimistic relative to real-world deployment.
- Baseline predictors only. Within-patient change over follow-up is not used.
- Median imputation, no hyperparameter tuning, no decision threshold chosen,
  no correction for multiple testing.
- No Fine-Gray model, so the Cox hazard ratios describe rates rather than probabilities.

## Author

**Anna Carolina Pousas Navarro**
Physician (Pediatrics, Allergy and Clinical Immunology), MSc in Systems Biology.
Research collaborator, Computational Systems Biology Laboratory, University of São Paulo.

[github.com/acpnavarro](https://github.com/acpnavarro)

## License

MIT. See [LICENSE](LICENSE).

If this notebook is useful in your teaching or your own work, a link back is appreciated
but not required.
