---
title: "Week 03 – Censored and Truncated Models"
course: MGT6203 – Data Analytics for Business
instructor: Lizhen Xu, Ph.D.
week: 3
topics:
  - Corner Solution Response
  - Censored Data
  - Truncated Data
  - Tobit Model
  - Truncated Regression Model
  - R Implementation (censReg, truncreg)
---

# Week 03 – Censored and Truncated Models

## Overview

This week continues the **Limited Dependent Variable** series. Where Week 02 covered binary outcomes (0 or 1), Week 03 deals with outcomes that are continuous but **restricted** — either because values are cut off at a boundary, or because some observations are missing entirely.

Three related but importantly different problems:

| Problem | What you observe | Model |
|---|---|---|
| Corner Solution Response | y is censored at a meaningful boundary (e.g. zero spending) | Tobit |
| Censored Data | y is cut off by an artificial data collection threshold | Tobit |
| Truncated Data | observations beyond a threshold are **completely missing** | Truncated Regression |

---

## Part 1 – The Full Family of Limited Dependent Variable Models

So far in the course:

- **Week 02** — Binary Response: y = 0 or 1 → Logit/Probit
- **Week 03** — Censored/Truncated: y is continuous but restricted → Tobit / Truncated Regression
- (Later) — Count Variables: y = 0, 1, 2, 3, ... → Poisson/Negative Binomial

All of these share the same core problem: using **OLS on the raw y values produces biased estimates** because the data structure violates the assumptions of linear regression.

---

## Part 2 – Corner Solution Response

### What is it?

The outcome variable y is naturally bounded — it **cannot go below (or above) some threshold**. Values within the valid range are continuous and fully observed. Values that would go beyond the boundary are "piled up" at the corner.

The key feature: **the corner value is meaningful** — it reflects a real economic or practical outcome, not a data collection limitation.

### Examples

| Variable | Boundary | Corner value means |
|---|---|---|
| Mobile data usage | ≥ 0 | User consumed no data that day |
| Charitable donations | ≥ 0 | Person donated nothing |
| Market/wallet share | [0, 1] | No share at all, or 100% |

### When does it need special treatment?

Only when the corner value (usually zero) accounts for a **significant portion** of the data. If almost everyone has a positive y, you can roughly ignore the issue and run OLS. When zeros are common, ignoring it causes **biased estimates** — OLS underestimates the true effect of x because it treats the zeros as regular data points rather than censored ones.

---

## Part 3 – The Tobit Model

The Tobit model handles both corner solution response and censored data using the **latent variable** framework (same idea as Week 02's Logit/Probit).

### Setup

$$y^* = \beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k + u, \quad u \sim N(0, \sigma^2)$$

$$y = \begin{cases} y^* & \text{if } y^* > 0 \\ 0 & \text{if } y^* \leq 0 \end{cases}$$

- $y^*$ is the **latent** (unobserved) variable — the "true" underlying value
- $y$ is what we **observe** in the data
- $u$ is the error term, assumed normally distributed with variance $\sigma^2$

### Key difference from Logit/Probit

In Logit/Probit, $\sigma^2$ was fixed to 1 (not identifiable from binary data). In Tobit, the **continuous positive values of y** provide enough information to also estimate $\sigma^2$ from the data. Both $\beta$ and $\sigma$ are estimated together.

### Why OLS fails

If you just run `lm()` on the observed y values and ignore the zeros being censored, OLS **underestimates the slope**. Intuitively: as x decreases, y hits zero and stays there — OLS sees "x keeps going down but y barely moves" and concludes the effect of x is weaker than it really is.

---

## Part 4 – Estimating the Tobit Model (MLE)

Like Logit/Probit, Tobit cannot be estimated with OLS. We use **Maximum Likelihood Estimation (MLE)**.

The likelihood has two parts depending on whether y > 0 or y = 0:

**For y > 0** (we observe the exact value):
$$f(y \mid x) = \frac{1}{\sigma} \phi\left(\frac{y - x\beta}{\sigma}\right)$$

**For y = 0** (censored at the corner):
$$P(y = 0 \mid x) = P(y^* \leq 0) = 1 - \Phi\left(\frac{x\beta}{\sigma}\right)$$

where $\Phi(\cdot)$ = CDF and $\phi(\cdot)$ = PDF of the standard normal distribution.

**Combined likelihood:**
$$\mathcal{L}(\beta, \sigma) = \prod_{y_i > 0} \sigma^{-1}\phi\left(\frac{y_i - x_i\beta}{\sigma}\right) \cdot \prod_{y_i = 0} \left[1 - \Phi\left(\frac{x_i\beta}{\sigma}\right)\right]$$

Solve: $(\hat{\beta}, \hat{\sigma}) = \arg\max_{\beta, \sigma} \mathcal{L}(\beta, \sigma)$ — numerically.

---

## Part 5 – Partial Effects in the Tobit Model

### Expected value of observed y

$$E(y \mid x) = P(y > 0 \mid x) \cdot E(y \mid y > 0, x) = \Phi\left(\frac{x\beta}{\sigma}\right) x\beta + \sigma\phi\left(\frac{x\beta}{\sigma}\right)$$

This is **non-linear** — it depends on $x\beta/\sigma$ through the normal CDF and PDF.

### Partial effect formula

$$\frac{\partial E(y \mid x)}{\partial x_j} = \beta_j \cdot \Phi\left(\frac{x\beta}{\sigma}\right)$$

### Two components of the effect

When $x_j$ increases, two things happen simultaneously:

1. **Extensive margin**: some people who were at zero (corner solution) may now move to positive y — i.e., $P(y > 0)$ increases
2. **Intensive margin**: for those already with $y > 0$, their y value also increases — i.e., $E(y \mid y > 0)$ increases

The Tobit partial effect captures both, but assumes they always move in the same direction (a limitation — see Part 7).

### How partial effects vary with x

| When $x\beta$ is... | $\Phi(x\beta/\sigma)$ | Partial effect |
|---|---|---|
| Very large | → 1 | → $\beta_j$ (same as OLS — censoring rarely matters) |
| Around zero | ≈ 0.5 | Moderate, somewhere between 0 and $\beta_j$ |
| Very small | → 0 | → 0 (almost everyone is at zero, x barely moves the needle) |

This is why Tobit partial effects follow an **S-shaped pattern** as x varies — unlike OLS where the effect is constant.

---

## Part 6 – Censored Data (Narrowly Defined)

### What is it?

Very similar structure to corner solution, but the **reason for censoring is different**. Here, the threshold is an **artificial data collection limitation** — not a meaningful economic constraint.

Examples:
- Survey records income as "≥ $100,000" for high earners (top-coding)
- Net worth below zero just recorded as "negative" with no exact value (bottom-coding)

In these cases, we know y is beyond the threshold but don't know the exact value.

### Key distinction from corner solution

| | Corner Solution | Censored Data |
|---|---|---|
| Why censored? | Real economic constraint | Artificial data collection limit |
| What is meaningful? | Observed y (including the zero) | Latent y* (the true unobserved value) |
| What do we want to predict? | E(y) — observed outcome | E(y*) — underlying true value |

### Estimation

Same Tobit model framework — estimated identically with MLE.

### Interpretation (simpler than corner solution!)

Because we care about $y^*$ not the observed $y$:

$$E(y^* \mid x) = x\beta$$

$$\frac{\partial E(y^* \mid x)}{\partial x_j} = \beta_j$$

The partial effect **is just the coefficient** — same as regular linear regression. Much simpler to interpret than the corner solution case.

---

## Part 7 – Limitations of the Tobit Model

The Tobit model assumes the extensive and intensive margins always move in the **same direction** — that is, whatever increases the probability of y > 0 also increases the amount of y given y > 0.

This is not always realistic. Example:
- As people **age**, the probability of buying life insurance may **increase**
- But the **amount** invested in life insurance may **decrease** with age

In this case, the two effects go in opposite directions, and Tobit averages them in a way that can be misleading.

The more general solution is a **two-part model** (Hurdle model) that models the two effects separately — beyond the scope of this course but worth being aware of.

---

## Part 8 – Truncated Data

### What is it?

Truncation is fundamentally different from censoring. In truncated data, **entire observations are missing** — we don't see the x variables OR the y variable for those individuals.

Think of it as: the data was collected only from a **non-random subset** of the population.

### Censored vs. Truncated — the critical distinction

| | Censored | Truncated |
|---|---|---|
| Do we observe x for all people? | ✅ Yes | ❌ No — x and y both missing |
| Do we know y is beyond the threshold? | ✅ Yes | ❌ No — no information at all |
| Still a random sample? | ✅ Yes | ❌ No — sample is selected |
| Bias if we use OLS on observed data? | Yes | Yes (sample selection bias) |

### Example

Studying the return on education for wages — but your dataset only includes people who are **employed**. Those who couldn't find work (effectively zero or negative wages) are completely missing from the data. Your sample is not random — it's selected based on the outcome variable.

---

## Part 9 – Truncated Regression Model

### Setup

$$y = \beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k + u, \quad u \sim N(0, \sigma^2)$$

$(x, y)$ is **missing entirely** if $y > c$ (or $y < c$ for lower-truncation).

### Likelihood

Because we only observe cases where $y \leq c$, the likelihood must be **conditioned** on observability:

$$g(y \mid x, c) = \frac{f(y - x\beta)}{F(c - x\beta)}$$

where $f(\cdot)$ = PDF and $F(\cdot)$ = CDF of $N(0, \sigma^2)$.

The denominator $F(c - x\beta)$ is the correction for sample selection — it scales the likelihood by the probability that this person would have been in the sample at all.

Estimated by MLE.

### Partial effects (simple!)

$$\frac{\partial E(y \mid x)}{\partial x_j} = \beta_j$$

The coefficient IS the partial effect — same as linear regression. The key is that $\hat{\beta}$ is now **corrected for sample selection bias** by the truncated model, whereas OLS on the truncated sample would give biased estimates.

---

## Part 10 – R Implementation

### Required packages

```r
install.packages("censReg")   # for Tobit model
install.packages("truncreg")  # for truncated regression

library(censReg)
library(truncreg)
```

### Simulate censored data (demo only — not needed for homework)

```r
set.seed(123)
n  <- 100
x  <- sort(runif(n, min = 2, max = 6))   # x ~ Uniform(2, 6)

# True parameters: β0 = -4, β1 = 1, σ = 1
ystar <- -4 + 1 * x + rnorm(n)           # latent y*

# Apply censoring at zero (corner solution)
y <- ystar
y[ystar <= 0] <- 0

plot(x, y, main = "Censored Data")
```

### Why OLS fails — demonstration

```r
ols <- lm(y ~ x)
summary(ols)
# β0 and β1 will be biased — slope underestimated
```

### Tobit model (corner solution / censored data)

```r
# censReg() with left = 0 means censored at zero from below
tobit <- censReg(y ~ x, left = 0, right = Inf, data = mydata)
summary(tobit)

# Extract β coefficients (first two; last is log(σ))
beta_hat <- coef(tobit)[1:2]

# Extract σ (stored as log(σ), so exponentiate)
sigma_hat <- exp(coef(tobit)[3])
cat("Estimated sigma:", sigma_hat, "\n")
```

> **Note on `censReg` output:** The last coefficient is `log(sigma)`, not `sigma` directly. Always exponentiate it: `exp(coef(tobit)["logSigma"])`.

### Marginal effects for Tobit

```r
# At the mean of all x variables (default)
margEff(tobit)

# At a specific x value — must pass as a vector with 1 first (for intercept)
margEff(tobit, xValues = c(1, 2.5))   # evaluate at x = 2.5
margEff(tobit, xValues = c(1, 6.0))   # evaluate at x = 6.0
```

> **Important:** `xValues` must be a vector, not a data frame — one point at a time. The first element must always be `1` to represent the intercept.

### Manual marginal effect calculation

```r
# Tobit partial effect formula: β_j × Φ(xβ / σ)
xb  <- coef(tobit)[1] + coef(tobit)[2] * x_val   # Xβ at point of interest
pe  <- coef(tobit)[2] * pnorm(xb / sigma_hat)     # β1 × Φ(Xβ/σ)
cat("Marginal effect at x =", x_val, ":", pe, "\n")
```

### Manual E(y) calculation

```r
# E(y|x) = Φ(xβ/σ) × xβ + σ × φ(xβ/σ)
ey <- pnorm(xb / sigma_hat) * xb + sigma_hat * dnorm(xb / sigma_hat)
```

### Truncated regression

```r
# direction = "left" means truncated from below at point = 0
# i.e., observations with y <= 0 are completely missing
trunc_model <- truncreg(y ~ x1 + x2, data = mydata, point = 0, direction = "left")
summary(trunc_model)

# Partial effects = coefficients directly (corrected for selection bias)
coef(trunc_model)
```

> **Key R functions summary:**
> - `censReg(formula, left=0, right=Inf, data)` → Tobit model
> - `margEff(tobit_result, xValues)` → marginal effects at a given point
> - `exp(coef(tobit)["logSigma"])` → recover σ from log(σ)
> - `pnorm()` → normal CDF $\Phi(\cdot)$; `dnorm()` → normal PDF $\phi(\cdot)$
> - `truncreg(formula, data, point=0, direction="left")` → truncated regression

---

## Part 11 – Python Implementation

### Libraries

```python
import numpy as np
import pandas as pd
from scipy.stats import norm
from scipy.optimize import minimize
import statsmodels.api as sm
```

### Tobit Model (manual MLE — no built-in statsmodels function)

```python
def tobit_loglik(params, y, X):
    """Negative log-likelihood for left-censored Tobit (censored at 0)."""
    beta  = params[:-1]
    sigma = np.exp(params[-1])   # parameterize as log(sigma) for stability

    xb    = X @ beta
    resid = (y - xb) / sigma

    # Likelihood contributions
    ll_pos  = norm.logpdf(resid[y > 0]) - np.log(sigma)   # y > 0: normal pdf
    ll_zero = norm.logcdf(-xb[y == 0] / sigma)            # y = 0: P(y* ≤ 0)

    return -(ll_pos.sum() + ll_zero.sum())   # return negative for minimization

# Set up X matrix with intercept
X = sm.add_constant(df[["x1", "x2"]].values)
y = df["y"].values

# Initial values
params0 = np.zeros(X.shape[1] + 1)   # betas + log(sigma)

result = minimize(tobit_loglik, params0, args=(y, X), method="BFGS")
beta_hat  = result.x[:-1]
sigma_hat = np.exp(result.x[-1])

print("Coefficients:", beta_hat)
print("Sigma:", sigma_hat)
```

### Tobit marginal effects

```python
# Partial effect: β_j × Φ(Xβ / σ) evaluated at point x0
x0  = np.array([1, mean_x1, mean_x2])   # include 1 for intercept
xb0 = x0 @ beta_hat

marginal_effects = beta_hat[1:] * norm.cdf(xb0 / sigma_hat)
print("Marginal effects:", marginal_effects)
```

### Expected y

```python
# E(y|x) = Φ(Xβ/σ) × Xβ + σ × φ(Xβ/σ)
ey = norm.cdf(xb0 / sigma_hat) * xb0 + sigma_hat * norm.pdf(xb0 / sigma_hat)
print("E(y|x):", ey)
```

> **Note:** Python does not have a dedicated Tobit package as clean as R's `censReg`. The manual MLE approach above is the standard way. There is also `linearmodels` and some third-party packages, but `censReg` in R is the recommended tool for this course.

---

## Summary

### The three problem types

| | Corner Solution | Censored Data | Truncated Data |
|---|---|---|---|
| Why restricted? | Real economic constraint | Artificial data limit | Sample selection |
| What do we observe? | y (including corner) and all x | y (capped) and all x | Only y ≤ c and corresponding x |
| Still random sample? | Yes | Yes | No |
| Model | Tobit | Tobit | Truncated Regression |
| Estimation | MLE via `censReg` | MLE via `censReg` | MLE via `truncreg` |
| Partial effect | $\beta_j \Phi(x\beta/\sigma)$ | $\beta_j$ (on $y^*$) | $\beta_j$ (corrected) |
| Constant partial effects? | No — varies with x | Yes | Yes |

### Why not just use OLS?

| Scenario | OLS problem |
|---|---|
| Corner solution / censored | Treats censored zeros as real data → underestimates slope |
| Truncated | Omits entire observations non-randomly → sample selection bias |

### Tobit partial effects vs. OLS

- OLS gives a **constant** slope — one number for all x values
- Tobit gives a **varying** marginal effect — as x grows, the effect of x approaches the true $\beta_j$; as x shrinks toward the censored region, the effect approaches zero
- OLS essentially gives you the **average** of the Tobit marginal effect across all x values — a crude approximation

---

## Key Formulas Cheat Sheet

| Quantity | Formula |
|---|---|
| Tobit model | $y^* = X\beta + u$, $y = \max(0, y^*)$ |
| $E(y \mid x)$ | $\Phi(x\beta/\sigma) \cdot x\beta + \sigma\phi(x\beta/\sigma)$ |
| Tobit partial effect | $\beta_j \cdot \Phi(x\beta/\sigma)$ |
| Censored data partial effect | $\beta_j$ (on latent $y^*$) |
| Truncated model likelihood | $g(y \mid x,c) = f(y-x\beta) / F(c-x\beta)$ |
| Truncated partial effect | $\beta_j$ (selection-corrected) |

*$\Phi$ = normal CDF, $\phi$ = normal PDF*
