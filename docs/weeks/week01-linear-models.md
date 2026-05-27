# Week 1 — Linear Models

**Prof. Lizhen Xu | MGT 6203**

---

## The Linear Regression Model

$$y_i = \beta_0 + \beta_1 x_{i1} + \beta_2 x_{i2} + \cdots + \beta_k x_{ik} + u_i$$

| Symbol | Meaning |
|---|---|
| $y_i$ | Dependent variable for observation $i$ |
| $x_{i1} \ldots x_{ik}$ | Independent variables for observation $i$ |
| $\beta_0$ | Intercept — baseline value of $y$ when all $x$'s = 0 |
| $\beta_1 \ldots \beta_k$ | Slope coefficients — the effect of each $x$ on $y$ |
| $u_i$ | Error term: $E(u_i \mid X) = 0$, $Var(u_i \mid X) = \sigma^2$ |

**Example:** `Price = β₀ + β₁(Age) + β₂(KM) + β₃(HP) + ... + u`

---

## OLS Estimation

### The Big Idea

We estimate the unknown $\beta$ coefficients by finding values that minimize the total squared prediction error across all observations:

$$\min_{\beta_0,\ldots,\beta_k} \sum_{i=1}^{n} \left(y_i - \beta_0 - \beta_1 x_{i1} - \cdots - \beta_k x_{ik}\right)^2$$

!!! note "Why squared?"
    Residuals can be positive or negative. Squaring them prevents positives and negatives from canceling each other out — every prediction error counts.

### In R

```r
model <- lm(Price ~ Age + KM + HP + Metallic + Automatic +
            CC + Doors + Gears + Weight, data = cars)
summary(model)
```

---

## Interpreting Coefficients

Each $\beta_j$ measures the **marginal / partial effect** of $x_j$ on $y$:

$$\Delta y = \beta_j \cdot \Delta x_j \quad \text{(holding all other variables constant)}$$

!!! tip "Ceteris Paribus"
    "Holding all other variables constant" is the key condition. This is what makes multiple regression powerful — it isolates each variable's individual effect.

**Example from homework:**

- Age coefficient = **−129.87 per month**
    - One year older (×12) → price drops ~**€1,558**
- KM coefficient = **−0.01463 per km**
    - 10,000 more km → price drops ~**€146**

---

## Goodness of Fit

### The Three Sums of Squares

| Measure | Formula | Meaning |
|---|---|---|
| **TSS** | $\sum(y_i - \bar{y})^2$ | Total variance in $y$ |
| **ESS** | $\sum(\hat{y}_i - \bar{y})^2$ | Variance explained by the model |
| **RSS** | $\sum(y_i - \hat{y}_i)^2$ | Variance left unexplained (residuals) |

$$TSS = ESS + RSS$$

### R²

$$R^2 = \frac{ESS}{TSS} = 1 - \frac{RSS}{TSS} \qquad (0 \leq R^2 \leq 1)$$

!!! info "Interpretation"
    $R^2 = 0.86$ means the model explains **86% of the variation** in Price. Larger = better fit.

!!! warning "Gotcha"
    Adding more variables **always increases R²**, even if those variables are useless. Never use R² alone to compare models with different numbers of predictors.

### Adjusted R²

$$\bar{R}^2 = 1 - \frac{RSS/(n-k-1)}{TSS/(n-1)}$$

Penalizes the model for each additional variable. If a variable doesn't earn its place, adjusted R² goes **down**. Use this when comparing models.

**Homework result:**

| Model | R² | Adjusted R² |
|---|---|---|
| Full (9 variables) | 0.86487 | 0.86390 |
| Reduced (6 variables) | 0.86485 | **0.86420** ✓ |

The reduced model wins on adjusted R² → simpler and better.

### In R

```r
summary(model)$r.squared       # R²
summary(model)$adj.r.squared   # Adjusted R²
```

---

## The t-Test

### Why It Matters

Our data is a random sample. A different sample would give different $\hat{\beta}$ values. So $\hat{\beta}_j$ is a random variable with its own variance:

$$Var(\hat{\beta}_j) = \frac{\sigma^2}{TSS_j \cdot (1 - R_j^2)}$$

The t-test asks: *Is $\hat{\beta}_j$ large enough relative to its uncertainty to be considered real?*

### The t-Statistic

$$t(\hat{\beta}_j) = \frac{\hat{\beta}_j}{se(\hat{\beta}_j)}$$

Follows a $t_{n-k-1}$ distribution under the null hypothesis $H_0: \beta_j = 0$.

### Step-by-Step

| Step | Action |
|---|---|
| **H₀** | $\beta_j = 0$ (variable has no effect) |
| **H₁** | $\beta_j \neq 0$ (variable has an effect) |
| **Critical value** | $c$ = $(1 - \alpha/2)$th percentile of $t_{n-k-1}$ |
| **Decision** | If $\lvert t \rvert > c$ → reject H₀ → significant |

### p-Value (equivalent approach)

$$p = 2 \cdot \Pr(t < -|t_{stat}|)$$

If $p < \alpha$ (e.g., 0.05 at 95% confidence) → significant.

!!! tip "Remember"
    - t-statistic: **larger** is better
    - p-value: **smaller** is better
    - Both methods always give the same conclusion

### In R

```r
# Critical value at 95% confidence
qt(0.975, df = model$df.residual)

# p-values
2 * pt(-abs(t_stats), df = model$df.residual)
```

---

## Multicollinearity & VIF

### The Problem

When independent variables are highly correlated with each other, $Var(\hat{\beta}_j)$ becomes very large → coefficients fail the t-test even if they have real effects.

### Variance Inflation Factor

$$VIF(\hat{\beta}_j) = \frac{1}{1 - R_j^2}$$

where $R_j^2$ is the R² from regressing $x_j$ on all other independent variables.

| VIF Value | Verdict |
|---|---|
| Close to 1 | No issue |
| 5–10 | Moderate concern |
| > 10 | Serious problem — consider removing a variable |

**Homework result:** All VIFs < 2.1 → no multicollinearity concern.

### In R

```r
library(car)
vif(model)
```

---

## Dummy Variables

### The Problem

Regression needs numbers. Categorical variables (color, city, gender) must be converted to 0/1 dummy variables.

### Variable Type Guide

| Type | Example | Treatment |
|---|---|---|
| Discrete numerical | Family size, # doors | Use as-is (if effect is linear) |
| Ordinal categorical | Education level | Create dummies if effect is non-linear |
| Nominal categorical | Color, city, ethnicity | Always create dummies |

### Two Levels (Binary)

```
xᵢ = 1 if female, 0 if male

→ β₀       = average outcome for males (baseline)
→ β₀ + β₁  = average outcome for females
→ β₁       = difference (female − male)
```

### Multiple Levels

Always create **(number of levels − 1)** dummies. One category = baseline.

```
Car color: red / white / black → create 2 dummies (red = baseline)

x₁ = 1 if white    → β₁ = white vs. red
x₂ = 1 if black    → β₂ = black vs. red
β₀ = average for red cars
```

!!! danger "Dummy Variable Trap"
    **Never include all levels as dummies.** Including all 3 colors (red + white + black) creates perfect multicollinearity — the model breaks. Always drop one category as the baseline.

---

## Quick Reference

| Concept | Formula | What It Tells You |
|---|---|---|
| OLS objective | $\min \sum (y_i - \hat{y}_i)^2$ | Find β's that minimize total squared error |
| Coefficient | $\Delta y = \beta_j \cdot \Delta x_j$ | Effect of 1-unit change in $x_j$ on $y$ |
| R² | $ESS/TSS$ | % of variation in Y explained by model |
| Adjusted R² | $1 - \frac{RSS/(n-k-1)}{TSS/(n-1)}$ | R² penalized for number of variables |
| t-statistic | $\hat{\beta}_j / se(\hat{\beta}_j)$ | Signal-to-noise ratio for each coefficient |
| p-value | $2 \cdot \Pr(t < -\lvert t_{stat}\rvert)$ | Probability of seeing this result by chance |
| VIF | $1/(1-R_j^2)$ | Multicollinearity detector |
