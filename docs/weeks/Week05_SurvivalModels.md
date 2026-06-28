---
title: "Week 05 – Survival Models"
course: MGT6203 – Data Analytics for Business
instructor: Lizhen Xu, Ph.D.
week: 5
topics:
  - Survival / Duration Analysis Overview
  - Duration, Survival Function, Hazard Function
  - Proportional Hazard Models
  - Weibull Distribution
  - Weibull Proportional Hazard Model
  - Coefficient Interpretation (β and δ)
  - Duration Dependence
  - R Implementation (survival package, survreg)
---

# Week 05 – Survival Models

## Overview

Survival analysis extends the binary response framework by adding **time information**. Instead of just asking *"did this event happen?"*, we now ask *"when did it happen — and what affects how long until it does?"*

The dependent variable now has **two pieces of information**:
1. Whether the observation is **right-censored** or not
2. The **time lapse** (duration) until the event or censoring

Common real-world applications:

| Context | Initial state | Event (exit) |
|---|---|---|
| Medicine | Patient receives treatment | Patient dies |
| Business | Customer signs up | Customer churns |
| Finance | Company is solvent | Company goes bankrupt |
| HR | Person becomes unemployed | Person finds a job |
| Manufacturing | Machine starts operating | Machine breaks down |
| Marketing | Customer makes first purchase | Customer makes repurchase |

---

## Part 1 – Key Concepts

### Duration T

**Duration** $T$ = the time lapse between when an individual **enters** an initial state and when they **exit** it.

- $T$ is a **random variable** — each person's actual duration is a realization of $T$
- For **uncensored** observations: $T$ is fully observed
- For **right-censored** observations: we only know the actual duration is **longer** than the observed time window

### Right Censoring

An observation is **right-censored** if the individual remains in the initial state until the end of the observation window. We know they haven't exited yet, but we don't know when they will.

> Example: A customer who hasn't made a repurchase by the end of the data collection period. We record the time since their last purchase but mark the record as censored.

---

## Part 2 – The Three Key Functions

All three functions below are **equivalent** — knowing one fully determines the others.

### 1. CDF and PDF of Duration

$$F(t) = \Pr(T \leq t) \quad \text{(probability that duration ends by time } t\text{)}$$

$$f(t) = \frac{dF}{dt}(t) \quad \text{(density of duration at time } t\text{)}$$

### 2. Survival Function

$$S(t) = 1 - F(t) = \Pr(T > t)$$

The probability of **surviving beyond** time $t$ — i.e., still being in the initial state at time $t$.

### 3. Hazard Function

$$\lambda(t) = \lim_{h \to 0} \frac{\Pr(t \leq T < t+h \mid T \geq t)}{h} = \frac{f(t)}{S(t)}$$

The **instantaneous probability density** of exiting the current state at time $t$, **conditional on having survived up to $t$**.

Intuitively: given that an individual has made it to time $t$, how likely are they to exit in the next tiny instant?

### Equivalence relationships

$$\lambda(t) = -\frac{d \ln S(t)}{dt}$$

$$S(t) = \exp\left[-\int_0^t \lambda(s)\, ds\right]$$

$$F(t) = 1 - \exp\left[-\int_0^t \lambda(s)\, ds\right]$$

$$f(t) = \lambda(t) \cdot \exp\left[-\int_0^t \lambda(s)\, ds\right]$$

> **Key insight:** Specifying the hazard function $\lambda(t)$ fully defines the entire survival model — the CDF, PDF, and survival function all follow from it.

### Simplest case: constant hazard

If $\lambda(t) \equiv \lambda$ (constant), the hazard never changes with time:
- Duration $T$ follows an **exponential distribution**: $f(t) = \lambda e^{-\lambda t}$
- Called **memoryless** — the probability of exiting doesn't depend on how long you've already been in the state

---

## Part 3 – Proportional Hazard (PH) Models

### Setup

Rather than specifying a free-form hazard, PH models impose a useful structure:

$$\lambda(t; X) = \kappa(X) \cdot \lambda_0(t)$$

- $\lambda_0(t)$: the **baseline hazard** — function of time only, same for all individuals
- $\kappa(X)$: the **coefficient term** — function of x variables only, scales the hazard differently per individual
- $\kappa(X) = e^{X\beta}$ (always positive, standard choice)

**Why "proportional"?** Because individuals' hazards are proportional to each other — they all share the same time pattern (baseline hazard) but are scaled by their x variable values.

### Likelihood function

For each observation $i$:

$$\ell(t_i \mid X_i, d_i) = f(t_i \mid X_i)^{d_i} \cdot [1 - F(t_i \mid X_i)]^{(1-d_i)}$$

- $d_i = 1$: uncensored → use PDF (exact duration observed)
- $d_i = 0$: censored → use $1-F$ = survival function (only know duration exceeded $t_i$)

Overall log-likelihood:
$$\mathcal{L}(\beta, \theta) = \sum_i \left\{ d_i \ln f(t_i \mid X_i) + (1-d_i) \ln[1 - F(t_i \mid X_i)] \right\}$$

Estimated by **MLE** — parameters include $\beta$ (x variable coefficients) and $\theta$ (baseline hazard parameters).

---

## Part 4 – Weibull Distribution

The Weibull distribution is the standard choice for the baseline hazard. It is defined over positive values ($t > 0$) and is very flexible.

### Formulas (standard parameterization)

$$f(t) = b^{-a} \cdot a \cdot t^{a-1} \cdot \exp(-b^{-a} \cdot t^a)$$

$$F(t) = 1 - \exp(-b^{-a} \cdot t^a)$$

$$\lambda(t) = b^{-a} \cdot a \cdot t^{a-1}$$

- $a > 0$: **shape parameter** — determines duration dependence
- $b > 0$: **scale parameter**

### Duration dependence — the role of shape parameter $a$

| Value of $a$ | Hazard over time | Duration dependence | Example |
|---|---|---|---|
| $a = 1$ | Constant | None (memoryless) | Reduces to exponential |
| $a > 1$ | Increasing in $t$ | **Positive** | Machine wear-out: older machines break more easily |
| $a < 1$ | Decreasing in $t$ | **Negative** | Post-surgery survival: if you make it past the critical period, you're increasingly likely to stay well |

### Checking Weibull fit

From the survival function: $\ln(-\ln S(t)) = a\ln(t) - a\ln(b)$

The **log-log of the survival function is linear in log of time**. To check whether Weibull fits your data, plot $\ln(-\ln \hat{S}(t))$ vs $\ln(t)$ — if it's approximately a straight line, Weibull is a good fit.

---

## Part 5 – Weibull Proportional Hazard Model

### Setup

Using the Weibull baseline hazard (with $b = 1$ for the baseline):

$$\lambda(t; X) = e^{X\beta} \cdot a \cdot t^{a-1}$$

**Important property**: The full hazard is also a Weibull hazard (with $b = e^{-X\beta/a}$). The Weibull distribution is **closed under proportional hazard specification** — which lets us derive the PDF and CDF in clean closed form.

### PDF and CDF

$$f(t) = e^{X\beta} \cdot a \cdot t^{a-1} \cdot \exp(-e^{X\beta} \cdot t^a)$$

$$F(t) = 1 - \exp(-e^{X\beta} \cdot t^a)$$

### Log-likelihood

$$\mathcal{L}(\beta, a) = \sum_i \left\{ d_i \ln f(t_i \mid X_i) + (1-d_i) \ln[1-F(t_i \mid X_i)] \right\}$$

Estimated by MLE — parameters are $\beta$ (x variable coefficients) and $a$ (Weibull shape).

---

## Part 6 – Interpreting Coefficients

### β coefficients — effect on hazard

Taking the log of the hazard:

$$\ln \lambda(t; X) = X\beta + \ln \lambda_0(t)$$

$$\beta_j = \frac{\partial}{\partial x_j} \ln \lambda(t; X) = \frac{\partial \lambda(t;X)}{\partial x_j} \cdot \frac{1}{\lambda(t;X)}$$

$\beta_j$ measures the **semi-elasticity of $x_j$ on the hazard** — a one-unit increase in $x_j$ changes the hazard by approximately $100\beta_j$ percent **at any given time $t$**.

> Example: $\beta_j = 0.17$ means a one-unit increase in $x_j$ increases the hazard by about **17%**.

> Note: higher hazard → shorter expected duration (the two move in opposite directions).

### δ coefficients — effect on expected duration

The Weibull model can also be written in **regression form**:

$$\ln T = X\delta + e$$

where $\delta_j = -\frac{\beta_j}{a}$ and $e$ is an error following a scaled extreme value distribution.

$\delta_j$ measures the **semi-elasticity of $x_j$ on the expected duration** — a one-unit increase in $x_j$ changes the expected duration by approximately $100\delta_j$ percent.

### Critical relationship between β and δ

$$\delta_j = -\frac{\beta_j}{a} \quad \Rightarrow \quad \beta_j = -\delta_j \cdot a$$

The signs are **always opposite** — consistent with the inverse relationship between hazard and duration.

### ⚠️ R's `survreg()` reports δ, NOT β

This is the most important gotcha in practice:

| What `survreg()` reports | What it means | How to get β |
|---|---|---|
| `scale` | $1/a$ | $a = 1/\text{scale}$ |
| Coefficients | $\delta_j$ | $\beta_j = -\delta_j \times a$ |

**Always transform** the `survreg()` output before interpreting.

---

## Part 7 – R Implementation

### Setup

```r
# install.packages("survival")
library(survival)

mydata <- read.csv("repurchase_data.csv")
str(mydata)
head(mydata)
```

### Estimate Weibull PH model

```r
# IMPORTANT: Surv() coding:
# event = 0 → right-censored
# event = 1 → NOT censored (opposite of how "censored" is usually coded in data)
# If your data has censored = 1 meaning censored, use (1 - censored)

weibull_model <- survreg(
  Surv(repurchase, 1 - censored) ~ promotion + income + gender,
  data = mydata,
  dist = "weibull"
)
summary(weibull_model)
```

### Transform to get β and a

```r
# Step 1: Extract shape parameter a = 1 / scale
scale_hat <- weibull_model$scale
a <- 1 / scale_hat
cat("Shape parameter a:", a, "\n")
cat("(log scale from summary =", log(scale_hat), "— verify matches output)\n")

# Step 2: Extract δ coefficients and convert to β
delta <- coef(weibull_model)   # these are δ, NOT β
beta  <- -delta * a            # β = -δ × a

cat("\nδ coefficients (from survreg):\n"); print(delta)
cat("\nβ coefficients (transformed):\n");  print(beta)
```

### Interpret β coefficients

```r
# β_j: one-unit increase in x_j changes hazard by ~100*β_j percent
# Positive β → higher hazard → shorter expected duration
# Negative β → lower hazard  → longer expected duration

cat("\nInterpretation: % change in hazard per unit increase in each x:\n")
print(round(beta * 100, 2))
```

### Duration dependence

```r
cat("Shape parameter a =", round(a, 4), "\n")
if (a > 1) {
  cat("→ a > 1: POSITIVE duration dependence (hazard increases over time)\n")
} else if (a < 1) {
  cat("→ a < 1: NEGATIVE duration dependence (hazard decreases over time)\n")
} else {
  cat("→ a = 1: No duration dependence (constant hazard, exponential distribution)\n")
}
```

### Plot the hazard function

```r
# Evaluate at mean x values
x_bar <- colMeans(mydata[, c("promotion", "income", "gender")])
xb    <- crossprod(c(1, x_bar), beta)   # Xβ at mean x values

# Hazard function: λ(t) = exp(Xβ) × a × t^(a-1)
curve(
  exp(c(xb)) * a * x^(a - 1),
  from = 0.1, to = 30,
  xlab = "Time (days)", ylab = "Hazard λ(t)",
  main = "Weibull Hazard Function"
)
```

### Plot the duration density function

```r
# Scale parameter b = exp(-Xβ/a) for the full Weibull
b_val <- exp(-c(xb) / a)

# Duration PDF using dweibull()
curve(
  dweibull(x, shape = a, scale = b_val),
  from = 0.1, to = 60,
  xlab = "Duration (days)", ylab = "Density f(t)",
  main = "Duration Density Function"
)

# Overlay histogram of observed (uncensored) durations
hist(mydata$repurchase[mydata$censored == 0],
     breaks = 30,
     freq   = FALSE,
     add    = TRUE,
     col    = NULL)
```

> **Key R functions summary:**
> - `library(survival)` → load survival package
> - `Surv(time, event)` → create survival object (`event`: 0 = censored, 1 = not)
> - `survreg(..., dist = "weibull")` → estimate Weibull PH model
> - `weibull_model$scale` → extract scale (= 1/a)
> - `coef(weibull_model)` → δ coefficients (must transform to get β)
> - `dweibull(x, shape = a, scale = b)` → Weibull PDF
> - `curve(...)` → plot a function over a range

---

## Part 8 – Python Implementation

### Libraries

```python
import numpy as np
import pandas as pd
from scipy.stats import weibull_min
from lifelines import WeibullPHFitter, KaplanMeierFitter
import matplotlib.pyplot as plt
```

### Estimate Weibull PH model

```python
df = pd.read_csv("repurchase_data.csv")

# lifelines expects: duration column + event observed column (1=event, 0=censored)
df["observed"] = 1 - df["censored"]   # reverse coding if needed

wph = WeibullPHFitter()
wph.fit(df, duration_col="repurchase", event_col="observed",
        formula="promotion + income + gender")
wph.print_summary()
```

### Extract parameters

```python
# lifelines reports β directly (log-hazard ratios)
beta_hat = wph.params_
print("β coefficients:\n", beta_hat)

# Shape parameter rho (= a in our notation)
rho = wph.rho_   # = a
print(f"\nShape parameter a (rho): {rho:.4f}")

if rho > 1:
    print("→ Positive duration dependence (hazard increases over time)")
elif rho < 1:
    print("→ Negative duration dependence (hazard decreases over time)")
else:
    print("→ No duration dependence (constant hazard)")
```

### Interpretation

```python
# β_j: ~% change in hazard per unit increase in x_j
print("\nApproximate % change in hazard per unit increase in each x:")
print((np.exp(beta_hat) - 1) * 100)   # exact % change = (e^β - 1) × 100
```

### Plot hazard and survival functions

```python
# Plot at mean covariate values
ax = wph.plot_hazard()
plt.title("Weibull Hazard Function")
plt.xlabel("Time (days)")
plt.show()

# Survival function
ax2 = wph.plot_survival_function()
plt.title("Survival Function")
plt.xlabel("Time (days)")
plt.show()
```

> **Note:** The `lifelines` package is the standard Python library for survival analysis. It reports β coefficients directly (unlike R's `survreg()` which reports δ), so no transformation is needed. Install with `pip install lifelines`.

---

## Summary

### The three functions and their relationship

| Function | Formula | Meaning |
|---|---|---|
| PDF $f(t)$ | $\lambda(t) \cdot S(t)$ | Probability density of duration ending at exactly $t$ |
| CDF $F(t)$ | $1 - S(t)$ | Probability duration has ended by time $t$ |
| Survival $S(t)$ | $\exp[-\int_0^t \lambda(s)ds]$ | Probability of surviving beyond $t$ |
| Hazard $\lambda(t)$ | $f(t)/S(t)$ | Instantaneous exit rate at $t$, given survival to $t$ |

### Weibull shape parameter decision rule

| $a$ | Duration dependence | Hazard shape |
|---|---|---|
| $a < 1$ | Negative | Decreasing |
| $a = 1$ | None | Constant (exponential) |
| $a > 1$ | Positive | Increasing |

### The survreg() coefficient transformation — must memorize

$$a = \frac{1}{\text{scale}}, \quad \beta = -\delta \times a$$

- `survreg()` gives you $\delta$ (effect on log duration) — **not** $\beta$
- You must multiply by $-a$ to get $\beta$ (effect on log hazard)
- Signs flip because hazard and duration move in opposite directions

---

## Key Formulas Cheat Sheet

| Quantity | Formula |
|---|---|
| Weibull hazard | $\lambda(t) = b^{-a} \cdot a \cdot t^{a-1}$ |
| Weibull PH hazard | $\lambda(t;X) = e^{X\beta} \cdot a \cdot t^{a-1}$ |
| Weibull PH PDF | $f(t) = e^{X\beta} \cdot a \cdot t^{a-1} \cdot \exp(-e^{X\beta} t^a)$ |
| Scale parameter $b$ | $b = e^{-X\beta/a}$ |
| β interpretation | $\beta_j \approx$ % change in hazard per unit $\Delta x_j$ |
| δ interpretation | $\delta_j \approx$ % change in expected duration per unit $\Delta x_j$ |
| Conversion | $\beta_j = -\delta_j \times a$; $a = 1/\text{scale}$ |
