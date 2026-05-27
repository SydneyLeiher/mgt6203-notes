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
    We can't know the true formula $f$ that connects inputs to outputs in the real world. Data analytics is the set of tools we use to *estimate* it from data as accurately as possible.

**Example:** Predicting a used car's price ($Y$) from its age, mileage, and horsepower ($X$). The true pricing formula $f$ is unknown — we estimate it from 1,264 real sales records.

---

## Why Estimate f?

There are two different reasons to build a model, and they lead to different choices:

| Goal | Description | Example |
|---|---|---|
| **Prediction** | Accurate guesses. $\hat{f}$ is a "black box." | Netflix recommendations |
| **Inference** | Understand the relationship. $\hat{f}$ must be interpretable. | Does ad spend actually increase sales? |

!!! warning "Key Trade-off"
    **More flexible models** → higher prediction accuracy, but harder to interpret.  
    **Simpler models** → easier to interpret, but may miss complex patterns.  
    You choose based on your goal.

---

## Types of Learning

### Supervised vs. Unsupervised

=== "Supervised"
    Has a $Y$ variable to predict. Goal: explore the relationship between $X$ and $Y$.

    - Examples: regression, classification
    - Used in: Weeks 1–7 of this course

=== "Unsupervised"
    No $Y$ variable. Goal: find hidden structure in $X$ itself.

    - Examples: clustering, topic modeling
    - Used in: Weeks 8–11 of this course

### Regression vs. Classification

| Type | Response Variable | Example |
|---|---|---|
| **Regression** | Numeric (continuous or discrete) | Predict salary, house price |
| **Classification** | Categorical | Predict yes/no, spam/not-spam |

---

## Bias-Variance Trade-off & Overfitting

### Sources of Prediction Error

$$E(Y - \hat{Y})^2 = \underbrace{Var(\hat{f}(X)) + [Bias(\hat{f}(X))]^2}_{\text{Reducible}} + \underbrace{Var(\varepsilon)}_{\text{Irreducible}}$$

| Component | What It Means |
|---|---|
| **Variance** | How much would the model change with a different training dataset? |
| **Bias** | Error from using an oversimplified model that misses real patterns |
| **Irreducible error** | Random noise in data — can never be eliminated |

!!! tip "The Goal"
    Minimize the **reducible** part (variance + bias²). The irreducible error is a hard floor — no model can beat it.

### Overfitting

!!! danger "Overfitting"
    When a model learns the **noise** in training data instead of the real pattern.  
    Result: fits training data perfectly, but predicts *new* data poorly.

**Example:** Memorizing all 100 practice exam questions vs. understanding the underlying concepts. Memorizing = overfitting. Understanding = generalizing.

---

## Quick Reference

| Concept | Key Idea |
|---|---|
| $Y = f(X) + \varepsilon$ | The master equation of data analytics |
| Prediction | Black-box accuracy |
| Inference | Interpretable relationships |
| Supervised | Has a Y to predict |
| Unsupervised | No Y — find structure in X |
| Regression | Y is a number |
| Classification | Y is a category |
| Overfitting | Learns noise, fails on new data |
| Bias-variance trade-off | Flexible = low bias, high variance |
