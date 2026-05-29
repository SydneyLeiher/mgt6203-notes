---
title: "Week 02 – Binary Response Models"
course: MGT6203 – Data Analytics for Business
instructor: Lizhen Xu, Ph.D.
week: 2
topics:
  - Linear Probability Models (LPM)
  - Logit Models
  - Probit Models
  - Model Evaluation (Confusion Matrix, PCP)
  - R Implementation
---

# Week 02 – Binary Response Models

## Overview

Binary response problems occur when the dependent variable **y** can only take two values: **0** or **1** (failure/success, no/yes). While this is technically a classification problem, binary response models are also regression-based — they predict the *probability* of success, which is a continuous numerical value.

Common real-world examples:
- Whether an applicant is admitted into a program
- Whether a customer defaults on a loan
- Whether a customer accepts a promotional offer
- Whether a customer churns

The three main model types covered this week:

| Model | Type | Estimation |
|---|---|---|
| Linear Probability Model (LPM) | Linear | OLS |
| Logit Model | Non-linear | MLE |
| Probit Model | Non-linear | MLE |

---

## Part 1 – Linear Probability Model (LPM)

### Setup

The LPM directly models the probability of success as a linear combination of predictors:

$$P(y = 1 \mid X) = \beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k$$

Because $y$ is binary, its expected value equals its probability of being 1:

$$E(y \mid X) = P(y=1 \mid X) \cdot 1 + P(y=0 \mid X) \cdot 0 = P(y=1 \mid X)$$

This is identical in structure to a standard linear regression. The LPM can therefore be estimated using **Ordinary Least Squares (OLS)** — just run `lm()` in R with a binary outcome variable.

The fitted value $\hat{y}$ = the **predicted probability of success**.

### Partial Effects (Marginal Effects)

The **partial effect** of variable $x_j$ measures how much a one-unit increase in $x_j$ changes the expected value of $y$, holding all other variables constant (ceteris paribus):

$$\frac{\partial E(y \mid X)}{\partial x_j} = \beta_j$$

In LPM, the coefficient **directly equals the partial effect** — a one-unit increase in $x_j$ changes the probability of success by $\beta_j$.

### Limitations

1. **Out-of-range predictions**: $\hat{y}$ can fall outside $[0,1]$, producing nonsensical probabilities (e.g., negative probability).
2. **Constant partial effects**: $\beta_j$ is fixed regardless of $x$ values — the model cannot capture diminishing returns or threshold effects.
3. Works reasonably well when $x$ values are **near the sample averages**; breaks down at extreme values.

---

## Part 2 – Latent Variable Model Framework

Logit and Probit models both arise from the **latent variable** framework. The key idea:

$$y^* = \beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k + e \equiv X\beta + e$$

$$y = \begin{cases} 1 & \text{if } y^* > 0 \\ 0 & \text{if } y^* \leq 0 \end{cases}$$

- $y^*$ is **unobserved** (latent) — think of it as the underlying propensity or utility
- $y$ is what we observe in the data
- $e$ is the unobserved error term; its distribution determines the model type

### Deriving P(y = 1)

$$P(y=1 \mid X) = P(y^* > 0) = P(e > -X\beta) = 1 - G(-X\beta) = G(X\beta)$$

where $G(\cdot)$ is the **CDF** of the error term distribution (using symmetry of the distribution in the last step).

**Key takeaway**: $P(y=1 \mid X) = G(X\beta)$, where $G$ is non-linear — so predicted probabilities are always in $[0,1]$.

---

## Part 3 – Logit Model

Assumes $e$ follows the **standard logistic distribution**.

### CDF (used in probability formula)

$$G(x) = \frac{e^x}{1 + e^x}$$

### PDF (used in partial effects)

$$g(x) = \frac{e^x}{(1+e^x)^2}$$

The logistic distribution is symmetric: $g(x) = g(-x)$.

### Probability of Success

$$P(y=1 \mid X) = \frac{\exp(\beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k)}{1 + \exp(\beta_0 + \beta_1 x_1 + \cdots + \beta_k x_k)}$$

This S-shaped function ensures $P \in (0,1)$ for all $X$.

### Log-Odds Interpretation

$$\frac{P(y=1 \mid X)}{1 - P(y=1 \mid X)} = e^{X\beta} \quad \Rightarrow \quad \ln\left(\frac{P(y=1)}{P(y=0)}\right) = X\beta$$

- **Odds** = ratio of probability of success to probability of failure
- $\beta_j$: a one-unit increase in $x_j$ **changes log-odds by $\beta_j$**, or equivalently **multiplies odds by $e^{\beta_j}$**

> Note: log-odds interpretation is valid but less intuitive than partial effects.

---

## Part 4 – Probit Model

Assumes $e$ follows the **standard normal distribution** $N(0,1)$.

### PDF

$$g(x) = \frac{1}{\sqrt{2\pi}} e^{-\frac{1}{2}x^2}$$

### CDF (no closed form — requires numerical integration)

$$G(x) = \int_{-\infty}^{x} \frac{1}{\sqrt{2\pi}} e^{-\frac{1}{2}z^2} dz$$

Because the normal CDF has no closed-form expression, the probit model is less mathematically convenient than logit — but both models typically produce very similar results in practice.

---

## Part 5 – Estimation: Maximum Likelihood (MLE)

Since logit/probit are non-linear, **OLS does not apply**. Instead, use **Maximum Likelihood Estimation**:

$$\hat{\beta} = \arg\max_{\beta} \mathcal{L}(\beta)$$

The likelihood function is:

$$\mathcal{L}(\beta) = \prod_{i: y_i=1} G(X_i\beta) \cdot \prod_{i': y_{i'}=0} [1 - G(X_i\beta)]$$

Intuition: find $\beta$ that makes:
- $\hat{P}(y_i = 1 \mid X_i) \to 1$ for observations where $y_i = 1$
- $\hat{P}(y_i = 1 \mid X_i) \to 0$ for observations where $y_i = 0$

No closed-form solution → solved numerically. In R, use `glm()`.

---

## Part 6 – Partial Effects in Logit/Probit

### Formula

$$\frac{\partial E(y \mid X)}{\partial x_j} = g(X\beta) \cdot \beta_j$$

where $g(\cdot)$ is the **PDF** of the error term distribution.

### Key Properties

| Property | Implication |
|---|---|
| Same sign as $\beta_j$ | Positive $\beta_j$ → positive partial effect |
| Changes with X values | Partial effects are not constant — addresses LPM limitation |
| Greatest when $X\beta \approx 0$ | Effect is largest in the middle of the distribution |
| Weakest when $X\beta$ is very large or small | Diminishing marginal effects at extremes |
| Comparable across models | Partial effects can be compared across logit/probit/LPM |
| $\beta_j$ NOT comparable across models | Logit and probit $\beta$'s are on different scales |

### S-Shaped Probability Curve

$P(y=1 \mid X) = G(X\beta)$ traces an **S-shape** (sigmoid) as $X\beta$ increases — slow growth at extremes, fastest growth in the middle. This is a realistic feature absent from LPM.

---

## Part 7 – Model Evaluation

### Goodness-of-Fit Measures

- **R²**: Standard measure, applicable to LPM
- **AIC / BIC**: Information criteria; preferred for MLE-estimated models (lower = better)
- **Percent Correctly Predicted (PCP)**: Most intuitive for binary outcomes

### Percent Correctly Predicted

**Step 1**: Predict probability of success for each observation:
- LPM: $\hat{y}_i = X_i\hat{\beta}$
- Logit/Probit: $\hat{y}_i = G(X_i\hat{\beta})$

**Step 2**: Choose threshold $c$ and assign binary predictions:
$$\tilde{y}_i = \begin{cases} 1 & \text{if } \hat{y}_i \geq c \\ 0 & \text{if } \hat{y}_i < c \end{cases}$$

Common threshold choices:
- $c = 0.5$ (default)
- $c = \bar{y}$ (fraction of successes in sample — preferred when outcomes are unbalanced)

**Step 3**: Construct the **confusion matrix**:

|  | Actual y = 1 | Actual y = 0 |
|---|---|---|
| **Predicted $\tilde{y}$ = 1** | True Positive (TP) | False Positive (FP) |
| **Predicted $\tilde{y}$ = 0** | False Negative (FN) | True Negative (TN) |

**Overall PCP**:
$$\text{PCP} = \frac{TP + TN}{TP + TN + FP + FN}$$

**Per-outcome PCP**:
$$\text{PCP}_{y=1} = \frac{TP}{TP + FN}, \quad \text{PCP}_{y=0} = \frac{TN}{TN + FP}$$

> ⚠️ High overall PCP can be misleading if the dataset is imbalanced. A model that always predicts the majority class can achieve high overall accuracy while completely failing on the minority class. Always check **per-outcome PCP**.

> ✅ A good model should predict the **least likely outcome** well too.

---

## Part 8 – R Implementation

### Data Preparation

```r
mydata <- read.csv("bank_data.csv")
mydata$loan <- as.factor(mydata$loan)   # convert binary categorical to factor
```

### Linear Probability Model

```r
lpm <- lm(Acquisition ~ ., data = mydata)
summary(lpm)

# Check for out-of-range predicted probabilities
yhat_lpm <- predict(lpm)
head(sort(yhat_lpm), 10)   # check smallest values (any < 0?)
tail(sort(yhat_lpm), 10)   # check largest values (any > 1?)
```

### Logit Model

```r
logit <- glm(Acquisition ~ ., data = mydata, family = binomial(link = "logit"))
summary(logit)
```

### Probit Model

```r
probit <- glm(Acquisition ~ ., data = mydata, family = binomial(link = "probit"))
summary(probit)
```

### Predict Probabilities and Binary Outcomes

```r
# Predicted probability of success (G(Xβ) for logit/probit)
yhat_prob <- predict(logit, type = "response")

# Choose threshold = fraction of actual successes
threshold <- mean(mydata$Acquisition)

# Assign binary predictions
yhat <- rep(1, nrow(mydata))
yhat[yhat_prob < threshold] <- 0
```

### Confusion Matrix and PCP

```r
conf_matrix <- table(yhat, mydata$Acquisition)
conf_matrix

# Overall PCP
pcp_overall <- sum(diag(conf_matrix)) / sum(conf_matrix)

# PCP by outcome
pcp_y1 <- conf_matrix[2,2] / sum(conf_matrix[,2])   # TP / (TP + FN)
pcp_y0 <- conf_matrix[1,1] / sum(conf_matrix[,1])   # TN / (TN + FP)
```

### Manual Predicted Probability (Logit Formula)

```r
beta <- coef(logit)
x_means <- colMeans(mydata[, 1:4])   # mean of numerical vars

# Manual calculation: exp(Xβ) / (1 + exp(Xβ))
xb <- beta[1] + beta[2]*x_means["age"] + beta[3]*x_means["income"] +
      beta[4]*x_means["homevalue"] + beta[5]*x_means["distance"] + beta[6]*1

prob_manual <- exp(xb) / (1 + exp(xb))

# Verify with predict()
newdata <- data.frame(age = x_means["age"], income = x_means["income"],
                      homevalue = x_means["homevalue"],
                      distance = x_means["distance"], loan = "Yes")
predict(logit, newdata = newdata, type = "response")
```

### Calculating Partial Effects

```r
# Linear combination Xβ (without response transformation)
xb_val <- predict(logit, newdata = newdata)

# Logit partial effect: g(Xβ) × β_j  where g = dlogis (logistic PDF)
g_xb <- dlogis(xb_val)
partial_effects_logit <- g_xb * coef(logit)[-1]   # exclude intercept

# LPM partial effects = coefficients directly
partial_effects_lpm <- coef(lpm)[-1]

# Compare side-by-side
cbind(LPM = partial_effects_lpm, Logit = partial_effects_logit)
```

> **Key R functions summary:**
> - `lm()` → LPM estimation
> - `glm(..., family = binomial(link = "logit"))` → Logit
> - `glm(..., family = binomial(link = "probit"))` → Probit
> - `predict(..., type = "response")` → $G(X\hat{\beta})$ predicted probabilities
> - `predict(...)` (no type) → $X\hat{\beta}$ linear index only
> - `dlogis()` → Logistic PDF; `dnorm()` → Normal PDF
> - `table()` → Confusion matrix

---

## Summary Comparison

| Feature | LPM | Logit | Probit |
|---|---|---|---|
| Probability guaranteed $\in [0,1]$ | ❌ | ✅ | ✅ |
| Partial effects constant | ✅ (always) | ❌ (vary with X) | ❌ (vary with X) |
| Estimation method | OLS | MLE | MLE |
| R function | `lm()` | `glm(link="logit")` | `glm(link="probit")` |
| Coefficient interpretation | Direct (= partial effect) | Log-odds (indirect) | No direct interpretation |
| Error distribution | Normal (assumed) | Logistic | Normal |
| Closed-form CDF | N/A | ✅ | ❌ |
| Works well near sample averages | ✅ | ✅ | ✅ |

---

## Key Formulas Cheat Sheet

| Quantity | LPM | Logit | Probit |
|---|---|---|---|
| $P(y=1 \mid X)$ | $X\beta$ | $\frac{e^{X\beta}}{1+e^{X\beta}}$ | $\Phi(X\beta)$ |
| Partial Effect | $\beta_j$ | $g(X\beta) \cdot \beta_j$ | $\phi(X\beta) \cdot \beta_j$ |
| PDF $g(\cdot)$ | — | $\frac{e^x}{(1+e^x)^2}$ | $\frac{1}{\sqrt{2\pi}}e^{-x^2/2}$ |

*Note: $\Phi$ = normal CDF, $\phi$ = normal PDF*
