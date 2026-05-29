# Week 1 — Overview of Data Analytics

**Prof. Lizhen Xu | MGT 6203**

---

## The Core Idea

Everything in data analytics comes down to one equation:

$$Y = f(X) + \varepsilon \quad \Rightarrow \quad \hat{Y} = \hat{f}(X)$$

| Symbol | Meaning |
|---|---|
| $Y$ | Dependent variable — what you're trying to predict |
| $X$ | Independent variables — your inputs / features |
| $\varepsilon$ | Error term — random noise you can't measure |
| $f$ | The true relationship between X and Y (unknown) |
| $\hat{f}$ | Our estimated version of $f$, learned from data |

!!! note "Plain English"
    We can never know the true formula $f$ that connects inputs to outputs in the real world. Data analytics is the set of tools we use to *estimate* it from data as accurately as possible.

**Example:** Predicting a used car's price ($Y$) from its age, mileage, and horsepower ($X$). The true pricing formula $f$ is unknown — we estimate it from historical sales data.

---

## Why Estimate f?

There are two different reasons to build a model, and they lead to different choices:

| Goal | Description | Example |
|---|---|---|
| **Prediction** | Accurate guesses. $\hat{f}$ is a "black box." | Netflix recommendations, fraud detection |
| **Inference** | Understand the relationship. $\hat{f}$ must be interpretable. | Does ad spend actually increase sales? |

!!! warning "Key Trade-off"
    **More flexible models** → higher prediction accuracy, but harder to interpret.  
    **Simpler models** → easier to interpret, but may miss complex patterns.  
    Choose based on your goal — do you need to *act* on the result or *explain* it?

**Real-world example:** A bank building a loan default model for regulators needs inference (explainability required by law). A bank optimizing click-through ads needs prediction (only accuracy matters).

---

## Types of Learning

### Supervised vs. Unsupervised

=== "Supervised"
    Has a $Y$ variable to predict. Goal: explore the relationship between $X$ and $Y$.

    - **Examples:** regression, classification, logistic regression, survival models
    - **Use when:** you have labeled historical data with a known outcome
    - **Real-world:** predicting customer churn, forecasting revenue, credit scoring

=== "Unsupervised"
    No $Y$ variable. Goal: find hidden structure in $X$ itself.

    - **Examples:** clustering, topic modeling, collaborative filtering
    - **Use when:** you want to discover patterns — no known outcome to train on
    - **Real-world:** customer segmentation, anomaly detection, recommendation systems

### Regression vs. Classification

| Type | Response Variable | Real-world Examples |
|---|---|---|
| **Regression** | Numeric — continuous or discrete | Forecast sales, predict house prices, estimate delivery time |
| **Classification** | Categorical — nominal or ordinal | Spam filter, churn prediction, medical diagnosis |

!!! tip "The boundary is blurry"
    Logistic regression is technically a *regression* model (it predicts a probability 0–1) but is used for *classification* tasks (yes/no). Don't get too hung up on the labels.

---

## Bias-Variance Trade-off & Overfitting

### Sources of Prediction Error

$$E(Y - \hat{Y})^2 = \underbrace{Var(\hat{f}(X)) + [Bias(\hat{f}(X))]^2}_{\text{Reducible}} + \underbrace{Var(\varepsilon)}_{\text{Irreducible}}$$

| Component | What It Means | How to Reduce |
|---|---|---|
| **Variance** | How much the model changes with different training data | Simpler model, more data, regularization |
| **Bias** | Error from oversimplifying — missing real patterns | More flexible model, add features |
| **Irreducible error** | Pure random noise — can never be eliminated | Nothing you can do |

!!! tip "The Goal"
    Minimize the **reducible** part (variance + bias²). You're always balancing the two — reducing bias tends to increase variance and vice versa.

### Overfitting

!!! danger "Overfitting"
    When a model learns the **noise** in training data instead of the real signal.  
    Result: fits training data perfectly, but predicts *new* data poorly.

**Example:** A model trained on 5 years of sales data that memorizes every holiday spike instead of learning the underlying seasonal trend. It scores perfectly in-sample but fails on next year's forecast.

**How to detect it:** Compare performance on training data vs. held-out test data. A big gap = overfitting.

---

## Implementation

=== "R"
    ```r
    # Load data
    df <- read.csv("data.csv")

    # Split into train/test (80/20)
    set.seed(42)
    train_idx <- sample(1:nrow(df), 0.8 * nrow(df))
    train <- df[train_idx, ]
    test  <- df[-train_idx, ]

    # Fit a supervised model (linear regression example)
    model <- lm(y ~ x1 + x2 + x3, data = train)

    # Predict on test set
    predictions <- predict(model, newdata = test)

    # Check for overfitting: compare train vs test error
    train_preds <- predict(model, newdata = train)
    train_rmse  <- sqrt(mean((train$y - train_preds)^2))
    test_rmse   <- sqrt(mean((test$y  - predictions)^2))

    cat("Train RMSE:", train_rmse, "\n")
    cat("Test RMSE: ", test_rmse,  "\n")
    # If test RMSE >> train RMSE, you're overfitting
    ```

=== "Python"
    ```python
    import pandas as pd
    import numpy as np
    from sklearn.linear_model import LinearRegression
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import mean_squared_error

    # Load data
    df = pd.read_csv("data.csv")

    # Define features (X) and target (y)
    X = df[["x1", "x2", "x3"]]
    y = df["y"]

    # Split into train/test (80/20)
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )

    # Fit a supervised model (linear regression example)
    model = LinearRegression()
    model.fit(X_train, y_train)

    # Predict on test set
    y_pred_test  = model.predict(X_test)
    y_pred_train = model.predict(X_train)

    # Check for overfitting: compare train vs test error
    train_rmse = np.sqrt(mean_squared_error(y_train, y_pred_train))
    test_rmse  = np.sqrt(mean_squared_error(y_test,  y_pred_test))

    print(f"Train RMSE: {train_rmse:.2f}")
    print(f"Test RMSE:  {test_rmse:.2f}")
    # If test RMSE >> train RMSE, you're overfitting
    ```

---

## Quick Reference

| Concept | Key Idea | Watch Out For |
|---|---|---|
| $Y = f(X) + \varepsilon$ | The master equation of data analytics | ε is irreducible — perfection is impossible |
| Prediction | Black-box accuracy | Hard to explain to stakeholders |
| Inference | Interpretable relationships | May sacrifice some accuracy |
| Supervised | Has a Y to predict | Needs labeled historical data |
| Unsupervised | No Y — find structure in X | Harder to validate results |
| Regression | Y is a number | |
| Classification | Y is a category | |
| Overfitting | Learns noise, fails on new data | Always validate on held-out test data |
| Bias-variance trade-off | Flexible = low bias, high variance | Find the sweet spot for your problem |