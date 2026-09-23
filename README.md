# Multilevel Multiple Imputation for a Longitudinal School Wellbeing Trial

R code from my **Mitacs Globalink Research Internship** at the **University of Manitoba** (summer 2018), on handling missing data in a clustered, longitudinal randomised trial run in schools.

---

## The problem

The trial measured student wellbeing at several time points, with students nested inside schools — so observations within a school are correlated, and that correlation (the ICC) is exactly what a multilevel analysis is trying to estimate.

Missing responses make this awkward:

- **Dropping incomplete cases** throws away most of the sample and biases estimates when data are not missing completely at random.
- **Standard single-level imputation** (e.g. plain `mice` with predictive mean matching) ignores the school clustering, which shrinks the between-school variance and biases the ICC downwards.

So the imputation model has to respect the same multilevel structure as the analysis model.

## What the code does

### `preprocessing/prepare_wellbeing_scores.Rmd`
Builds the analysis dataset from the raw trial export:

- recodes eight ordinal wellbeing items ("Rarely or Never" … "Everyday or almost everyday") to numeric, keeping "Don't know" distinct from missing;
- derives several mean-score variants per time point, differing in how "Don't know" is treated (as a value, or as missing);
- reshapes the per-time-point columns into long format, one row per student per time point, ready for a mixed model.

### `imputation/multilevel_imputation.Rmd`
The analysis itself:

1. **Complete-case baseline** — `lme(meanscore ~ case * Time, random = ~1 | student)` with `nlme`, then richer models adding sex and the prosocial / total-difficulties scores, compared by likelihood-ratio test and AIC.
2. **Single-level imputation** — `mice` with predictive mean matching, 30 imputed datasets, 10 iterations, with a hand-built predictor matrix so that design variables are never imputed from the cluster id, plus convergence plots.
3. **Multilevel imputation** — imputation that treats the cluster variable as a grouping factor, so between-cluster variance survives the imputation.
4. **Comparison** — ICC and fixed effects across complete-case, single-level and multilevel imputation, pooled with Rubin's rules.

## Running it

```r
install.packages(c("tidyverse", "mice", "nlme", "multilevel", "pan", "micemd"))
```

Then knit either notebook in RStudio.

> **The trial data is not included.** The notebooks read `project11.csv` / `longdata.csv`, which hold per-student records from a school trial and are not mine to publish. The code is here for the method, not to be re-run as is.

## Context

Selected for the Mitacs Globalink Research Internship (top 800 of ~10,000 applicants across 10 countries). The work improved data quality for the trial analysis while preserving the intra-class correlation needed for the mixed-effects models.

## License

MIT — see [LICENSE](LICENSE).
