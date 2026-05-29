---
title: "MGT6203 – Data Analytics for Business"
instructor: Lizhen Xu, Ph.D.
institution: Georgia Tech
notes_version: "1.0"
last_updated: "2026-05-29"
---

# MGT6203 – Data Analytics for Business
### Course Notes Index

> Comprehensive notes compiled from lecture slides, transcripts, and R demos.
> Each week folder contains a detailed `.md` notes file.

---

## Course Structure

### Part 1 – Econometric & Statistical Models

| Week | Topic | Notes | Status |
|---|---|---|---|
| 00 | Introduction to R | — | — |
| 01 | Linear Models | [Week01_LinearModels.md](./Week01_LinearModels.md) | ✅ (slides in project) |
| **02** | **Binary Response Models** | [Week02_BinaryResponseModels.md](./Week02_BinaryResponseModels.md) | ✅ Complete |
| 03 | Censored / Truncated Data Models | — | 🔜 |
| 04 | Count Data Models | — | 🔜 |
| 05 | Survival Models | — | 🔜 |
| 06 | Discrete Choice Models | — | 🔜 |
| 07 | Causal Inference: Instrumental Variables | — | 🔜 |

### Part 2 – Machine Learning Methods

| Week | Topic | Notes | Status |
|---|---|---|---|
| 08 | Cluster Analysis + Collaborative Filtering | — | 🔜 |
| 09 | Text Mining: Overview + Topic Modeling | — | 🔜 |
| 10 | Neural Networks | — | 🔜 |
| 11 | Deep Learning | — | 🔜 |

---

## Week 02 – Binary Response Models (Quick Reference)

**Key Models:** Linear Probability Model · Logit · Probit

### Core Formulas

| Model | $P(y=1 \mid X)$ | Partial Effect |
|---|---|---|
| LPM | $X\beta$ | $\beta_j$ |
| Logit | $\frac{e^{X\beta}}{1+e^{X\beta}}$ | $g(X\beta) \cdot \beta_j$ |
| Probit | $\Phi(X\beta)$ | $\phi(X\beta) \cdot \beta_j$ |

### R Cheat Sheet

```r
# LPM
lm(y ~ ., data = df)

# Logit
glm(y ~ ., data = df, family = binomial(link = "logit"))

# Probit
glm(y ~ ., data = df, family = binomial(link = "probit"))

# Predicted probabilities
predict(model, type = "response")

# Confusion matrix
table(predicted, actual)

# Logistic PDF (for partial effects)
dlogis(predict(logit_model, newdata = x0))
```

---

## References

| Source | Details |
|---|---|
| **ISLR** (Primary) | *An Introduction to Statistical Learning*, James et al., 2nd Ed., Springer 2021 |
| **MLBA** (Supplementary) | *Machine Learning for Business Analytics*, Shmueli et al., 2nd Ed., Wiley 2023 |
| **Wooldridge** (Supplementary) | *Introductory Econometrics: A Modern Approach*, 7th Ed., Cengage 2019 |
| **The Book of R** (R Reference) | Davies, T.M., No Starch Press, 2016 |

---

## Notes on Key Concepts

### Binary Response vs. Regression vs. Classification

Binary outcomes are technically *classification* problems, but binary response models approach them through *regression* (predicting probability). The boundary between classification and regression is blurry here — logistic regression can be called either.

### Why Logit over Probit?

Both come from the same latent variable framework; the only difference is the error term distribution:
- **Logit** → logistic distribution → closed-form CDF → mathematically convenient ✅
- **Probit** → normal distribution → no closed-form CDF → numerically solved ⚠️

In practice, both give nearly identical results.

### Why Partial Effects > Coefficients for Interpretation

- LPM: $\beta_j$ = partial effect directly
- Logit/Probit: $\beta_j$ has the right *sign* but wrong *scale* for interpretation
- Partial effects are **comparable across all model types**; raw $\beta$ coefficients are **not**

### Model Evaluation Pitfall

A model predicting the majority class every time can have high overall PCP on imbalanced data. Always check **per-outcome accuracy** — a good model must also predict the *minority/rare class* reasonably well.
