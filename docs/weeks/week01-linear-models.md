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

Find the $\beta$ values that minimize the total squared prediction error across all observations:

$$\min_{\beta_0,\ldots,\beta_k} \sum_{i=1}^{n} \left(y_i - \hat{y}_i\right)^2$$

!!! note "Why squared?"
    Residuals can be positive (underpredicted) or negative (overpredicted). Squaring prevents them from canceling out — every error counts regardless of direction.

### Implementation

=== "R"
    ```r
    # Fit a linear regression model
    # lm(formula, data): formula uses ~ to mean "is modeled by"
    model <- lm(y ~ x1 + x2 + x3, data = df)

    # Full summary: coefficients, std errors, t-stats, p-values, R²
    summary(model)

    # Extract just the coefficients
    coef(model)

    # Get fitted values (ŷ) and residuals
    fitted_vals   <- fitted(model)      # predicted y for each observation
    residual_vals <- residuals(model)   # actual y minus predicted y
    ```

=== "Python"
    ```python
    import pandas as pd
    import statsmodels.formula.api as smf
    from sklearn.linear_model import LinearRegression

    # --- Option 1: statsmodels (gives full stats output like R) ---
    model = smf.ols("y ~ x1 + x2 + x3", data=df).fit()
    print(model.summary())          # coefficients, p-values, R², etc.
    print(model.params)             # just the coefficients
    print(model.fittedvalues)       # fitted ŷ values
    print(model.resid)              # residuals

    # --- Option 2: scikit-learn (better for pure prediction pipelines) ---
    from sklearn.linear_model import LinearRegression
    X = df[["x1", "x2", "x3"]]
    y = df["y"]
    model = LinearRegression().fit(X, y)
    print(model.coef_)              # slope coefficients
    print(model.intercept_)         # intercept
    predictions = model.predict(X)
    ```

!!! tip "Which to use in practice?"
    Use **statsmodels** when you need p-values, confidence intervals, and inference (explaining *why*).
    Use **scikit-learn** when you're building a prediction pipeline and accuracy is all that matters.

---

## Interpreting Coefficients

Each $\beta_j$ measures the **marginal / partial effect** of $x_j$ on $y$:

$$\Delta y = \beta_j \cdot \Delta x_j \quad \text{(holding all other variables constant)}$$

!!! tip "Ceteris Paribus"
    "Holding all other variables constant" is the key condition. This isolates each variable's individual effect — that's the power of multiple regression over just looking at correlations.

**Scaling tip:** If your variable isn't in "1-unit" increments that make sense, scale the coefficient:

| Variable | Coefficient | Meaningful interpretation |
|---|---|---|
| Age (months) | −129.87/month | × 12 → **−€1,558 per year** |
| KM (per km) | −0.01463/km | × 10,000 → **−€146 per 10k km** |

=== "R"
    ```r
    # Extract a specific coefficient and scale it
    coef(model)["Age"] * 12        # annual effect
    coef(model)["KM"]  * 10000     # effect of 10,000 km
    ```

=== "Python"
    ```python
    # With statsmodels
    model.params["Age"] * 12       # annual effect
    model.params["KM"]  * 10000    # effect of 10,000 km

    # With scikit-learn (coefficients are in the same order as your column list)
    feature_names = ["Age", "KM", "HP"]
    coef_dict = dict(zip(feature_names, model.coef_))
    coef_dict["Age"] * 12
    ```

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

$R^2 = 0.86$ means the model explains **86% of the variation** in $y$. Larger = better fit.

!!! warning "Gotcha"
    Adding more variables **always increases R²**, even useless ones. Never use R² alone to compare models with different numbers of predictors.

### Adjusted R²

$$\bar{R}^2 = 1 - \frac{RSS/(n-k-1)}{TSS/(n-1)}$$

Penalizes the model for each extra variable. Adding a useless variable lowers adjusted R². Use this when comparing models.

=== "R"
    ```r
    summary(model)$r.squared        # R²
    summary(model)$adj.r.squared    # Adjusted R²

    # Manual calculation
    y     <- df$y
    y_hat <- fitted(model)
    y_bar <- mean(y)

    TSS <- sum((y - y_bar)^2)
    ESS <- sum((y_hat - y_bar)^2)
    RSS <- sum((y - y_hat)^2)

    R2 <- ESS / TSS
    ```

=== "Python"
    ```python
    # statsmodels
    print(model.rsquared)           # R²
    print(model.rsquared_adj)       # Adjusted R²

    # scikit-learn
    from sklearn.metrics import r2_score
    r2 = r2_score(y, model.predict(X))

    # Manual calculation
    import numpy as np
    y_bar = np.mean(y)
    TSS   = np.sum((y - y_bar)**2)
    RSS   = np.sum((y - model.predict(X))**2)
    R2    = 1 - RSS / TSS
    ```

---

## The t-Test

### Why It Matters

Our data is a random sample. A different sample would give different $\hat{\beta}$ values — so $\hat{\beta}_j$ is a random variable with its own variance:

$$Var(\hat{\beta}_j) = \frac{\sigma^2}{TSS_j \cdot (1 - R_j^2)}$$

The t-test asks: *Is this coefficient large enough relative to its uncertainty to be real, or could it just be sampling noise?*

### The t-Statistic & p-Value

$$t(\hat{\beta}_j) = \frac{\hat{\beta}_j}{se(\hat{\beta}_j)} \qquad \text{follows } t_{n-k-1} \text{ under } H_0: \beta_j = 0$$

$$p = 2 \cdot \Pr(t < -|t_{stat}|)$$

| Decision rule | Condition |
|---|---|
| Reject $H_0$ (significant) | $\lvert t \rvert >$ critical value **or** $p < \alpha$ |
| Fail to reject (not significant) | $\lvert t \rvert \leq$ critical value **or** $p \geq \alpha$ |

!!! tip "Remember"
    - t-statistic: **larger absolute value** = stronger evidence
    - p-value: **smaller** = stronger evidence
    - At 95% confidence: critical value ≈ **1.96**, alpha = **0.05**

=== "R"
    ```r
    # summary() gives t-stats and p-values automatically
    summary(model)

    # Manual critical value at 95% confidence
    alpha      <- 0.05
    t_critical <- qt(1 - alpha/2, df = model$df.residual)

    # Manual p-values
    coef_table    <- summary(model)$coefficients
    t_stats       <- coef_table[, "t value"]
    p_vals_manual <- 2 * pt(-abs(t_stats), df = model$df.residual)
    ```

=== "Python"
    ```python
    # statsmodels gives everything automatically
    print(model.summary())          # includes t-stats and p-values

    # Access individually
    print(model.tvalues)            # t-statistics for each coefficient
    print(model.pvalues)            # p-values for each coefficient
    print(model.conf_int())         # 95% confidence intervals

    # Manual critical value
    from scipy import stats
    alpha      = 0.05
    df_resid   = model.df_resid
    t_critical = stats.t.ppf(1 - alpha/2, df=df_resid)

    # Manual p-values
    t_stats       = model.tvalues
    p_vals_manual = 2 * stats.t.cdf(-abs(t_stats), df=df_resid)
    ```

---

## Multicollinearity & VIF

### The Problem

When independent variables are highly correlated with each other, $Var(\hat{\beta}_j)$ inflates → coefficients fail the t-test even when they have real effects. Hard to tell which variable is actually doing the work.

### Variance Inflation Factor

$$VIF(\hat{\beta}_j) = \frac{1}{1 - R_j^2}$$

where $R_j^2$ = R² from regressing $x_j$ on all other independent variables.

| VIF Value | Verdict |
|---|---|
| ~1 | No issue — variable is independent of others |
| 5–10 | Moderate concern — worth investigating |
| > 10 | Serious problem — consider removing a variable |

**Real-world example:** Including both square footage and number of rooms in a house price model — they're highly correlated, causing multicollinearity. Drop one or combine them.

=== "R"
    ```r
    library(car)

    # Calculate VIF for all predictors
    vif(model)

    # Manual VIF for one variable (e.g., x1)
    aux_model <- lm(x1 ~ x2 + x3 + x4, data = df)
    R2_x1     <- summary(aux_model)$r.squared
    VIF_x1    <- 1 / (1 - R2_x1)
    ```

=== "Python"
    ```python
    from statsmodels.stats.outliers_influence import variance_inflation_factor
    import pandas as pd

    X = df[["x1", "x2", "x3", "x4"]]

    # Calculate VIF for each feature
    vif_data = pd.DataFrame({
        "Feature": X.columns,
        "VIF": [variance_inflation_factor(X.values, i)
                for i in range(X.shape[1])]
    })
    print(vif_data)

    # Manual VIF for one variable (x1)
    import statsmodels.formula.api as smf
    aux = smf.ols("x1 ~ x2 + x3 + x4", data=df).fit()
    VIF_x1 = 1 / (1 - aux.rsquared)
    ```

---

## Dummy Variables

### The Problem

Regression needs numbers. Categorical variables must be converted to 0/1 dummy (indicator) variables.

### Variable Type Guide

| Type | Example | Treatment |
|---|---|---|
| Discrete numerical | Number of doors, family size | Use as-is if effect is linear |
| Ordinal categorical | Education level, satisfaction rating | Create dummies if effect isn't linear |
| Nominal categorical | Color, city, department | Always create dummies |

### Binary (2 levels)

```
x = 1 if female, 0 if male  →  β₁ = average difference (female − male)
```

### Multiple Levels

Always create **(levels − 1)** dummies. One category becomes the **baseline**.

```
Region: North / South / East / West  →  3 dummies (North = baseline)
β₁ = South vs. North
β₂ = East  vs. North
β₃ = West  vs. North
```

!!! danger "Dummy Variable Trap"
    **Never include all levels.** Including all 4 regions creates perfect multicollinearity — the model breaks. Always drop one as the baseline reference group.

=== "R"
    ```r
    # R handles factors automatically — it drops the first level as baseline
    df$region <- as.factor(df$region)  # convert to factor
    model     <- lm(y ~ x1 + region, data = df)
    summary(model)

    # To change the baseline category:
    df$region <- relevel(df$region, ref = "North")
    ```

=== "Python"
    ```python
    import pandas as pd

    # Option 1: pandas get_dummies (drop_first removes one level automatically)
    df_encoded = pd.get_dummies(df, columns=["region"], drop_first=True)

    # Option 2: statsmodels handles C() notation like R factors
    import statsmodels.formula.api as smf
    model = smf.ols("y ~ x1 + C(region)", data=df).fit()
    # C(region) automatically creates dummies and drops the first level

    # To set a specific baseline:
    model = smf.ols("y ~ x1 + C(region, Treatment('North'))", data=df).fit()
    ```

---

## Full Workflow

=== "R"
    ```r
    library(car)

    # 1. Load and inspect data
    df <- read.csv("data.csv")
    str(df)
    summary(df)

    # 2. Fit the full model
    model_full <- lm(y ~ x1 + x2 + x3 + x4, data = df)
    summary(model_full)

    # 3. Check for multicollinearity
    vif(model_full)

    # 4. Identify significant variables (p < 0.05)
    coef_table <- summary(model_full)$coefficients
    sig_vars   <- rownames(coef_table)[coef_table[, "Pr(>|t|)"] < 0.05]

    # 5. Refit with only significant variables
    model_reduced <- lm(y ~ x1 + x3, data = df)  # example
    summary(model_reduced)

    # 6. Compare models with adjusted R²
    summary(model_full)$adj.r.squared
    summary(model_reduced)$adj.r.squared
    ```

=== "Python"
    ```python
    import pandas as pd
    import statsmodels.formula.api as smf
    from statsmodels.stats.outliers_influence import variance_inflation_factor

    # 1. Load and inspect data
    df = pd.read_csv("data.csv")
    print(df.info())
    print(df.describe())

    # 2. Fit the full model
    model_full = smf.ols("y ~ x1 + x2 + x3 + x4", data=df).fit()
    print(model_full.summary())

    # 3. Check for multicollinearity
    X = df[["x1", "x2", "x3", "x4"]]
    vif_df = pd.DataFrame({
        "Feature": X.columns,
        "VIF": [variance_inflation_factor(X.values, i)
                for i in range(X.shape[1])]
    })
    print(vif_df)

    # 4. Identify significant variables (p < 0.05)
    sig_vars = model_full.pvalues[model_full.pvalues < 0.05].index.tolist()
    print("Significant:", sig_vars)

    # 5. Refit with only significant variables
    model_reduced = smf.ols("y ~ x1 + x3", data=df).fit()  # example
    print(model_reduced.summary())

    # 6. Compare models with adjusted R²
    print("Full adj R²:   ", model_full.rsquared_adj)
    print("Reduced adj R²:", model_reduced.rsquared_adj)
    ```

---

## Quick Reference

| Concept | Formula | What It Tells You |
|---|---|---|
| OLS objective | $\min \sum (y_i - \hat{y}_i)^2$ | Find β's that minimize total squared error |
| Coefficient | $\Delta y = \beta_j \cdot \Delta x_j$ | Effect of 1-unit change in $x_j$ on $y$, all else equal |
| R² | $ESS/TSS$ | % of variation in Y explained by model |
| Adjusted R² | $1 - \frac{RSS/(n-k-1)}{TSS/(n-1)}$ | R² penalized for number of variables — use for model comparison |
| t-statistic | $\hat{\beta}_j / se(\hat{\beta}_j)$ | Signal-to-noise ratio — larger = more significant |
| p-value | $2 \cdot \Pr(t < -\lvert t\rvert)$ | Probability result is due to chance — smaller = more significant |
| VIF | $1/(1-R_j^2)$ | Multicollinearity detector — keep below 5 |