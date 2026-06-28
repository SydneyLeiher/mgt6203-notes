---
title: "Week 04 – Count Data Models"
course: MGT6203 – Data Analytics for Business
instructor: Lizhen Xu, Ph.D.
week: 4
topics:
  - Poisson Distribution
  - Poisson Regression Model
  - Coefficient Interpretation & Partial Effects
  - Overdispersion
  - Negative Binomial Model
  - Gamma-Poisson Mixture
  - AIC / BIC
  - R Implementation (glm, MASS)
---

# Week 04 – Count Data Models

## Overview

Count data models are the next type of **Limited Dependent Variable** model. The dependent variable y takes **non-negative integer values** (0, 1, 2, 3, ...) — especially when those values are relatively small.

Examples: number of store visits per month, number of social media posts, number of insurance claims, number of children in a family.

**Why not just use OLS or Tobit?**
- If count values span a wide range (hundreds/thousands), treating them as continuous is a reasonable approximation
- If zeros are common and values span a moderate range, Tobit may work
- When values are mostly small (0, 1, 2, 3), approximating integers as continuous is a **poor fit** — we need a proper count data model

The two models covered this week:

| Model | Assumption | Best when |
|---|---|---|
| Poisson Regression | Var(y) = E(y) | No overdispersion |
| Negative Binomial | Var(y) > E(y) | Overdispersion present |

---

## Part 1 – The Poisson Distribution

The foundation of Poisson regression is the **Poisson distribution** — a discrete probability distribution for non-negative integer outcomes.

### PMF (Probability Mass Function)

$$P(y = k \mid \lambda) = \frac{\lambda^k e^{-\lambda}}{k!}, \quad k = 0, 1, 2, \ldots \quad (\lambda > 0)$$

- $\lambda$ is the single parameter — must be positive
- $e \approx 2.718$ (Euler's number)
- $k!$ = factorial of k (note: $0! = 1$)

### Example probabilities

$$P(y=0) = e^{-\lambda}, \quad P(y=1) = \lambda e^{-\lambda}, \quad P(y=2) = \frac{1}{2}\lambda^2 e^{-\lambda}$$

### Key properties

$$E(y) = \lambda, \quad Var(y) = \lambda$$

> The **mean equals the variance** — this is the defining constraint of a Poisson distribution, and the source of its main limitation (overdispersion).

As $\lambda$ increases, the distribution shifts right and spreads out. For small $\lambda$ (e.g., $\lambda = 1$), the distribution is heavily right-skewed with most mass at 0 and 1. For large $\lambda$ (e.g., $\lambda = 10$), it becomes more symmetric and bell-shaped.

---

## Part 2 – Poisson Regression Model

### Setup

In Poisson regression, the **mean parameter $\lambda$ is modeled as a function of the x variables**:

$$E(y \mid X) = \lambda = \exp(\beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k) = \exp(X\beta)$$

**Why exp(Xβ) and not just Xβ?**
- $X\beta$ can be negative, but $\lambda$ must be positive
- $\exp(X\beta)$ is always positive regardless of the sign of $X\beta$
- The exponential function preserves ordering (increasing function)

So y follows a Poisson distribution with $\lambda = \exp(X\beta)$:

$$P(y = k \mid X) = \frac{\exp(X\beta)^k \cdot e^{-\exp(X\beta)}}{k!}$$

### Estimation via MLE

Because of the non-linear structure, OLS cannot be used. We use **Maximum Likelihood Estimation (MLE)**.

Log-likelihood for one observation:
$$\ell_i(\beta) = y_i X_i\beta - \exp(X_i\beta)$$

Overall log-likelihood (sum over all observations):
$$\mathcal{L}(\beta) = \sum_i \left[ y_i X_i\beta - \exp(X_i\beta) \right]$$

$$\hat{\beta} = \arg\max_\beta \mathcal{L}(\beta)$$

Solved numerically. In R: `glm(..., family = poisson)`.

---

## Part 3 – Interpreting Poisson Coefficients

### Log-linear structure

Taking the log of both sides of the Poisson mean equation:

$$\ln[E(y \mid X)] = \beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k = X\beta$$

This is a **log-linear model** — the log of the expected outcome is linear in x.

### Direct interpretation of β

Taking the partial derivative with respect to $x_j$:

$$\frac{\partial \ln[E(y \mid X)]}{\partial x_j} = \beta_j \quad \Rightarrow \quad \frac{\partial E(y \mid X)}{\partial x_j} \cdot \frac{1}{E(y \mid X)} = \beta_j$$

This means: $\beta_j$ measures the **approximate percentage change** in the expected count when $x_j$ increases by one unit.

> Example: if $\beta_j = 0.25$, a one-unit increase in $x_j$ increases $E(y)$ by approximately **25%**.

This is the clearest and most intuitive way to communicate Poisson results.

### Partial (marginal) effects

The absolute change in E(y) per one-unit increase in $x_j$:

$$\frac{\partial E(y \mid X)}{\partial x_j} = \exp(X\beta) \cdot \beta_j$$

Like Logit/Probit, partial effects **vary with x values** — not constant.

### Average Partial Effect (APE)

To get an overall summary across all observations:

$$\text{APE}_j = \frac{1}{n} \sum_{i=1}^n \exp(X_i\hat{\beta}) \cdot \hat{\beta}_j = \bar{y} \cdot \hat{\beta}_j$$

The last equality follows from the MLE first-order condition — the average predicted value equals the average actual y. So:

$$\text{APE}_j = \bar{y} \cdot \hat{\beta}_j$$

This is a very convenient shortcut — just multiply the coefficient by the sample mean of y.

---

## Part 4 – Overdispersion

### The problem

Poisson assumes $Var(y) = E(y)$. In real data, it's common to observe:

$$Var(y) \gg E(y) \quad \text{(overdispersion)}$$

Less common: $Var(y) < E(y)$ (underdispersion).

If overdispersion is present and you use a Poisson model, your **standard errors will be too small** and your inference will be misleading (false significance).

### Estimating the dispersion parameter

Define $\sigma^2$ such that $Var(y \mid X) = \sigma^2 E(y \mid X)$:
- $\sigma^2 = 1$ → Poisson assumption holds
- $\sigma^2 > 1$ → overdispersion
- $\sigma^2 < 1$ → underdispersion

Estimate $\sigma^2$ from data after fitting the Poisson model:

$$\hat{\sigma}^2 = \frac{1}{n-k-1} \sum_{i=1}^n \frac{(y_i - \hat{y}_i)^2}{\hat{y}_i}$$

where $\hat{y}_i = \exp(X_i\hat{\beta})$ and $k$ = number of x variables.

If $\hat{\sigma}^2$ is notably greater than 1 → overdispersion → consider Negative Binomial.

---

## Part 5 – Negative Binomial Model

### Motivation

The Negative Binomial model extends Poisson by adding **individual random effects** ($\nu_i$) to the mean parameter:

$$E(y_i \mid X_i) = \nu_i \cdot \mu_i = \nu_i \cdot \exp(X_i\beta)$$

- $\mu_i = \exp(X_i\beta)$: the deterministic part (same as Poisson)
- $\nu_i$: individual random coefficient — unobservable, varies person to person — captures extra heterogeneity not explained by x variables

### Gamma distribution for $\nu_i$

$\nu_i$ must be positive, so we use the **Gamma distribution**. Setting $\alpha = \beta = \theta > 0$:

$$f(\nu; \theta) = \frac{\theta^\theta}{\Gamma(\theta)} \nu^{\theta-1} e^{-\theta\nu}$$

$$E(\nu) = 1, \quad Var(\nu) = \frac{1}{\theta}$$

$\theta$ is the **dispersion parameter**:
- Small $\theta$ → large $Var(\nu)$ → significant overdispersion
- Large $\theta$ → small $Var(\nu)$ → little overdispersion
- $\theta \to \infty$ → $Var(\nu) \to 0$ → all $\nu_i = 1$ → reduces to Poisson

### Deriving the Negative Binomial distribution

Integrating out the unobservable $\nu_i$ from the Poisson probability (Gamma-Poisson mixture):

$$P(y = k) = \int_0^\infty \frac{(\nu\mu)^k e^{-\nu\mu}}{k!} \cdot \frac{\theta^\theta}{\Gamma(\theta)} \nu^{\theta-1} e^{-\theta\nu} d\nu$$

After integration (closed-form thanks to the Gamma-Poisson conjugacy):

$$P(y = k) = \frac{\Gamma(k+\theta)}{\Gamma(\theta)k!} \left(\frac{\theta}{\theta+\mu}\right)^\theta \left(\frac{\mu}{\theta+\mu}\right)^k \sim NegBinom(\theta, \mu)$$

Substituting $\mu = \exp(X\beta)$ gives the full **Negative Binomial Regression Model**.

### Properties

$$E(y \mid X) = \mu = \exp(X\beta)$$

$$Var(y \mid X) = \mu + \frac{1}{\theta}\mu^2 > E(y \mid X)$$

The negative binomial is **always overdispersed** by design. The extra term $\frac{1}{\theta}\mu^2$ represents the additional variance from individual heterogeneity.

### Convergence to Poisson

As $\theta \to \infty$: $Var(y) \to \mu = E(y)$ → becomes Poisson. So Poisson is a **special case** of Negative Binomial.

---

## Part 6 – Poisson vs. Negative Binomial

| | Poisson | Negative Binomial |
|---|---|---|
| $E(y \mid X)$ | $\exp(X\beta)$ | $\exp(X\beta)$ |
| $Var(y \mid X)$ | $\exp(X\beta)$ | $\exp(X\beta) + \frac{1}{\theta}\exp(X\beta)^2$ |
| Dispersion parameter | None (fixed at 1) | $\theta$ (estimated) |
| Handles overdispersion | ❌ | ✅ |
| Reduces to Poisson when | Always | $\theta \to \infty$ |
| R function | `glm(..., family=poisson)` | `glm.nb()` from MASS |

### When is Poisson still OK despite overdispersion?

Because both models share the same $E(y \mid X) = \exp(X\beta)$, the Poisson coefficient estimates are still **consistent** even when overdispersion is present. Using Poisson MLE when the Poisson distribution isn't strictly correct is called **Quasi-MLE (QMLE)**.

- If you only care about **expected values** (mean of y) → Poisson is fine
- If you care about the **full distribution** (e.g., probability of y = 0, or y ≥ 5) → use Negative Binomial to properly account for overdispersion

---

## Part 7 – Goodness of Fit: AIC and BIC

For MLE-estimated models, R² is not the right metric. Use **AIC** and **BIC** instead.

### Formulas

$$AIC = 2k - 2\ln(\hat{L})$$

$$BIC = k\ln(n) - 2\ln(\hat{L})$$

- $\hat{L}$: maximized likelihood value
- $k$: number of parameters in the model
- $n$: number of observations

### Key properties

- Both based on maximized log-likelihood — better fit = higher $\hat{L}$ = lower AIC/BIC
- Both **penalize model complexity** — adding parameters increases k and increases the penalty
- BIC imposes a **heavier penalty** ($k\ln(n)$ vs $2k$) and tends to favor simpler models
- **Lower is better** for both
- Use to compare competing models (e.g., Poisson vs. Negative Binomial)

### Decision rule

If Negative Binomial has lower AIC/BIC than Poisson → NB is a better fit → overdispersion is significant enough to matter.

If AIC/BIC are similar → little evidence of overdispersion → Poisson is adequate.

---

## Part 8 – R Implementation

### Data and setup

```r
# install.packages("MASS")   # for negative binomial
library(MASS)

mydata <- read.csv("mall_visit_data.csv")
str(mydata)

# Quick histogram of count variable
hist(mydata$visit, breaks = 0:10,
     main = "Distribution of Monthly Visits",
     xlab = "Number of Visits")
```

### Poisson regression

```r
# Use glm() with family = poisson — same function as Logit/Probit
poisson_model <- glm(visit ~ ., data = mydata, family = poisson)
summary(poisson_model)

# Predicted expected counts
yhat <- predict(poisson_model, type = "response")   # = exp(Xβ)
```

### Interpreting coefficients

```r
# Direct interpretation: β_j ≈ % change in E(y) per unit increase in x_j
coef(poisson_model)

# Average Partial Effect: ȳ × β_j
y_bar <- mean(mydata$visit)
ape <- y_bar * coef(poisson_model)[-1]   # exclude intercept
print(ape)
```

### Checking for overdispersion

```r
y    <- mydata$visit
yhat <- predict(poisson_model, type = "response")
n    <- nrow(mydata)
k    <- length(coef(poisson_model)) - 1   # number of x variables

sigma2_hat <- sum((y - yhat)^2 / yhat) / (n - k - 1)
cat("Dispersion statistic σ²:", sigma2_hat, "\n")
# σ² ≈ 1 → no overdispersion; σ² >> 1 → overdispersion
```

### Negative Binomial regression

```r
# glm.nb() from MASS package
nb_model <- glm.nb(visit ~ ., data = mydata)
summary(nb_model)

# Theta (dispersion parameter) — shown at bottom of summary
# Large theta → little overdispersion → close to Poisson
# Small theta → significant overdispersion
theta_hat <- nb_model$theta
cat("Estimated theta:", theta_hat, "\n")
```

### Compare AIC and BIC

```r
cbind(
  Poisson = c(AIC = AIC(poisson_model), BIC = BIC(poisson_model)),
  NegBinom = c(AIC = AIC(nb_model),     BIC = BIC(nb_model))
)
# Lower = better fit
```

### Plot probability distributions (Poisson vs NB)

```r
# Evaluate at mean x values
x_bar <- colMeans(mydata[, 2:6])   # adjust column indices to x variables

# Xβ at mean x values
xb_poisson <- crossprod(c(1, x_bar), coef(poisson_model))
xb_nb      <- crossprod(c(1, x_bar), coef(nb_model))

k_vals <- 0:10

# Poisson probabilities
p_poisson <- dpois(k_vals, lambda = exp(xb_poisson))

# Negative Binomial probabilities
p_nb <- dnbinom(k_vals, mu = exp(xb_nb), size = theta_hat)

# Plot
plot(k_vals, p_nb, pch = 16, col = "blue",
     xlab = "Count (k)", ylab = "P(y = k)",
     main = "Poisson vs. Negative Binomial PMF")
points(k_vals, p_poisson, pch = 1, col = "red")
legend("topright", legend = c("Negative Binomial", "Poisson"),
       pch = c(16, 1), col = c("blue", "red"))
```

> **Key R functions summary:**
> - `glm(..., family = poisson)` → Poisson regression
> - `glm.nb(...)` from MASS → Negative Binomial regression
> - `predict(..., type = "response")` → $\exp(X\hat{\beta})$ predicted counts
> - `AIC()`, `BIC()` → goodness-of-fit comparison
> - `dpois(k, lambda)` → Poisson PMF
> - `dnbinom(k, mu, size)` → Negative Binomial PMF (size = $\theta$)

---

## Part 9 – Python Implementation

### Libraries

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm
import statsmodels.formula.api as smf
from scipy.stats import poisson, nbinom
import matplotlib.pyplot as plt
```

### Poisson regression

```python
df = pd.read_csv("mall_visit_data.csv")

# Poisson via statsmodels
poisson_model = smf.poisson("visit ~ income + gender + distance + discount + promo",
                             data = df).fit()
print(poisson_model.summary())

# Predicted expected counts
yhat = poisson_model.predict()   # = exp(Xβ)
```

### Coefficient interpretation and APE

```python
beta = poisson_model.params

# Direct: β_j ≈ % change in E(y) per unit increase in x_j
print("Coefficients (≈ % effect on E(y)):\n", beta)

# Average Partial Effect: ȳ × β_j
y_bar = df["visit"].mean()
ape   = y_bar * beta.drop("Intercept")
print("\nAverage Partial Effects:\n", ape)
```

### Overdispersion check

```python
y    = df["visit"].values
n    = len(y)
k    = len(beta) - 1   # number of x variables

sigma2_hat = np.sum((y - yhat)**2 / yhat) / (n - k - 1)
print(f"Dispersion statistic σ²: {sigma2_hat:.4f}")
# Close to 1 → Poisson OK; >> 1 → use Negative Binomial
```

### Negative Binomial regression

```python
# statsmodels NegativeBinomial
nb_model = smf.negativebinomial("visit ~ income + gender + distance + discount + promo",
                                 data = df).fit()
print(nb_model.summary())

# Alpha parameter (= 1/θ in statsmodels parameterization)
alpha_hat = nb_model.params["alpha"]
theta_hat = 1 / alpha_hat
print(f"Theta (dispersion): {theta_hat:.4f}")
# Large theta → little overdispersion; small theta → significant overdispersion
```

> **Note on parameterization:** statsmodels NegativeBinomial uses `alpha = 1/θ`. Small alpha → little overdispersion (close to Poisson). This is the **opposite** of R's theta convention.

### Compare AIC and BIC

```python
print(pd.DataFrame({
    "Poisson":  {"AIC": poisson_model.aic, "BIC": poisson_model.bic},
    "NegBinom": {"AIC": nb_model.aic,      "BIC": nb_model.bic}
}))
```

### Plot PMF comparison

```python
x_bar = df.drop(columns=["visit"]).mean()
x0    = np.concatenate([[1], x_bar.values])

xb_p  = x0 @ poisson_model.params.values
xb_nb = x0 @ nb_model.params.drop("alpha").values

k_vals    = np.arange(0, 11)
p_poisson = poisson.pmf(k_vals, mu = np.exp(xb_p))
p_nb      = nbinom.pmf(k_vals,
                        n = theta_hat,
                        p = theta_hat / (theta_hat + np.exp(xb_nb)))

plt.figure(figsize=(8, 4))
plt.plot(k_vals, p_nb,      "bo-", label="Negative Binomial")
plt.plot(k_vals, p_poisson, "rs--", label="Poisson")
plt.xlabel("Count (k)")
plt.ylabel("P(y = k)")
plt.title("Poisson vs. Negative Binomial PMF")
plt.legend()
plt.tight_layout()
plt.show()
```

---

## Summary

### The count data problem in context

| Week | Problem | y type | Model |
|---|---|---|---|
| 02 | Binary Response | 0 or 1 | Logit / Probit |
| 03 | Corner Solution / Censored | Continuous ≥ 0 | Tobit |
| **04** | **Count Data** | **0, 1, 2, 3, ...** | **Poisson / Negative Binomial** |

### Poisson vs. Negative Binomial decision guide

Start with Poisson. Calculate $\hat{\sigma}^2$:
- $\hat{\sigma}^2 \approx 1$ → Poisson is adequate
- $\hat{\sigma}^2 \gg 1$ → use Negative Binomial

Also check $\hat{\theta}$ from Negative Binomial:
- Large $\hat{\theta}$ (e.g., > 10) → little overdispersion → Poisson fine
- Small $\hat{\theta}$ → significant overdispersion → Negative Binomial needed

Confirm with AIC/BIC comparison.

---

## Key Formulas Cheat Sheet

| Quantity | Poisson | Negative Binomial |
|---|---|---|
| $E(y \mid X)$ | $\exp(X\beta)$ | $\exp(X\beta)$ |
| $Var(y \mid X)$ | $\exp(X\beta)$ | $\exp(X\beta) + \frac{1}{\theta}\exp(X\beta)^2$ |
| $\beta_j$ interpretation | ~% change in $E(y)$ | ~% change in $E(y)$ |
| Partial effect | $\exp(X\beta) \cdot \beta_j$ | $\exp(X\beta) \cdot \beta_j$ |
| APE | $\bar{y} \cdot \hat{\beta}_j$ | $\bar{y} \cdot \hat{\beta}_j$ |
| Overdispersion | Cannot handle | Built-in via $\theta$ |
| Reduces to Poisson when | Always | $\theta \to \infty$ |
