---
title: "Week 06 – Discrete Choice Models"
course: MGT6203 – Data Analytics for Business
instructor: Lizhen Xu, Ph.D.
week: 6
topics:
  - Discrete Choice Framework Overview
  - The Choice Set (mutually exclusive, exhaustive, finite)
  - Random Utility Model (RUM)
  - Choice Probabilities
  - Multinomial Logit Model
  - Type I (Gumbel) Extreme Value Distribution
  - Binary Logit as a Special Case
  - Model Identification
  - Model Specification (general and basic)
  - MLE Estimation
  - Marginal Effects (self and cross)
  - R Implementation (mlogit package)
---

# Week 06 – Discrete Choice Models

## Overview

Discrete choice models **extend the binary response framework** to allow the outcome variable to take **more than two** possible values — in fact, any finite number of categories.

- **Binary response:** choose one out of **two** alternatives (the outcome is 0 or 1)
- **Discrete choice:** choose one out of **multiple** alternatives

In the language of discrete choice, the possible categories of the dependent variable are called **alternatives**. Binary response models are therefore just a **special case** of discrete choice models with only two alternatives.

The most commonly applied form is the **multinomial logit model**, obtained by imposing a particular assumption on the distribution of the error term.

Common real-world applications:

| Context | Decision | Alternatives |
|---|---|---|
| Marketing | Which brand to buy | Brand A, B, C, … for the same product type |
| Transportation | Daily commute mode | Drive, bus, bike, … |
| Education | Which program to attend | Different colleges / analytics programs |
| Energy | Which heating system to install | Gas central, electric central, heat pump, … |

> Discrete choice models are heavily studied and applied in **economics and marketing**, where understanding and influencing consumer behavior is central.

---

## Part 1 – The Choice Set

The **choice set** is the set of alternatives each decision maker faces. For discrete choice models, it must satisfy three properties:

1. **Mutually exclusive** — an individual can choose only one alternative
2. **Exhaustive** — every possible alternative is included in the set
3. **Finite** — the number of alternatives is a finite integer

Together, properties 1 and 2 mean each individual chooses **one and only one** alternative.

### Relaxing the requirements

These conditions look restrictive but mostly are not, because simple transformations restore them:

| Apparent problem | Fix |
|---|---|
| Alternatives not mutually exclusive (can pick two at once) | Create a **new combined alternative** for the bundle |
| Can't observe all alternatives in the data | Add an **"outside option"** / "anything else" alternative |
| Too many alternatives to estimate | **Group** alternatives into a few categories and model the group choice |

> The one **real** limitation is finiteness — discrete choice models cannot handle an **infinite** number of options, and are best suited to a **relatively small** number of alternatives.

---

## Part 2 – Random Utility Model (RUM)

Discrete choice models are built on the **Random Utility Model**. Individual $i$'s utility from choosing alternative $j$ is:

$$U_{ij} = V_{ij} + \varepsilon_{ij}$$

**Watch the two-dimensional subscripts:**
- $i$ indexes **individuals** (observations in the sample)
- $j$ indexes **alternatives** (the categories the outcome can take)

The utility has two parts:

| Part | Symbol | Meaning |
|---|---|---|
| **Representative (deterministic) utility** | $V_{ij} = V(X_{ij};\beta)$ | The part explained by **observed** attributes; usually modeled as a linear combination $X_{ij}\beta$ |
| **Random error term** | $\varepsilon_{ij}$ | The combined effect of **unobserved** factors; modeled as a random variable |

> **Subtle but important:** $\varepsilon_{ij}$ is unobserved **to the analyst**, not to the individual. Individual $i$ sees all their own $\varepsilon_{ij}$ values and uses them to decide. We just can't observe them, so we treat them as random.

### Choice probabilities

Each individual picks the alternative giving the **highest utility**. So the probability individual $i$ chooses $j$ is:

$$P_{ij} = \Pr(U_{ij} > U_{ik}, \ \forall k \neq j)$$

$$= \Pr(V_{ij} + \varepsilon_{ij} > V_{ik} + \varepsilon_{ik}, \ \forall k \neq j)$$

$$= \Pr(\varepsilon_{ik} - \varepsilon_{ij} < V_{ij} - V_{ik}, \ \forall k \neq j)$$

> **Key takeaway:** the choice probability depends on the **distribution of the error terms** (the differences in errors vs. the differences in representative utilities).

---

## Part 3 – Multinomial Logit Model

### Distribution of the error term

The error distribution determines the model:

| Error distribution | Model |
|---|---|
| Extreme value | **Logit** |
| Multivariate normal | **Probit** |
| More complex | Nested logit, mixed logit, … |

For the **multinomial logit**, we assume each $\varepsilon_{ij}$ is **i.i.d. extreme value** (Independent and Identically Distributed) across all alternatives $j$.

### Type I (Gumbel) Extreme Value Distribution

$$f(\varepsilon) = e^{-\varepsilon} e^{-e^{-\varepsilon}} \qquad F(\varepsilon) = e^{-e^{-\varepsilon}}$$

Properties:

| Property | Value / Note |
|---|---|
| Mean $E[\varepsilon]$ | $\gamma$ = **Euler's constant** $\approx 0.5772$ — **not zero!** |
| Variance $\text{Var}(\varepsilon)$ | $\dfrac{\pi^2}{6} \approx 1.6$ |
| Shape | Bell-shaped, **slightly skewed right**, **slightly fatter tails**, **not symmetric about 0** |
| In practice | **Empirically indistinguishable from normal** |

> **Why use it instead of normal?** Because it produces a **neat closed-form expression** for the choice probabilities — the same reason the logistic distribution is used for binary logit. The closed form is the key analytical payoff.

### The closed-form choice probability

Treating $\varepsilon_{ij}$ as given, the independence of the $\varepsilon_{ik}$ lets us write the probability as a product of CDFs, then integrate over $\varepsilon_{ij}$ using its density. After simplification:

$$\boxed{P_{ij} = \frac{e^{V_{ij}}}{\sum_{k} e^{V_{ik}}}}$$

**Intuition** (ignore the exponentials and treat them as an increasing transformation of $V$): the probability of choosing $j$ is the **share of its (transformed) utility relative to the sum of all alternatives' utilities**.

- $V_{ij}$ (own utility) ↑ → $P_{ij}$ ↑
- Competing $V_{ik}$ (denominator) ↑ → $P_{ij}$ ↓

**Guaranteed to be a valid probability:**
- Always **> 0** (exponentials are positive)
- Always **< 1** (numerator is one term inside the denominator sum)
- All alternatives' probabilities **sum to 1** (numerators add to the denominator) — consistent with each individual choosing exactly one alternative

---

## Part 4 – Binary Logit as a Special Case

Binary logit is multinomial logit with **two** alternatives, normalizing one alternative's representative utility to zero ($V_{i0} = 0$, $V_{i1} = V$):

$$\Pr(y_i = 1) = \frac{e^{V}}{1 + e^{V}}$$

Because $e^{0} = 1$, the denominator collapses to $1 + e^{V}$ — exactly the binary logit formula.

Moreover, the **difference of two i.i.d. extreme value variables** $\varepsilon^{*} = \varepsilon_j - \varepsilon_k$ follows the **logistic distribution**:

$$F(\varepsilon^{*}) = \frac{e^{\varepsilon^{*}}}{1 + e^{\varepsilon^{*}}}$$

> So the binomial logit model is literally a 2-alternative discrete choice model where the error is the **difference** between the two alternatives' errors. Discrete choice **naturally extends** the binary response framework.

---

## Part 5 – Model Identification

Two things are **not identified** from the data and require normalization.

### 1. Only *differences* in utility matter (absolute level is irrelevant)

Adding a common constant to **all** alternatives' utilities doesn't change which is largest, so it doesn't change any choice. Consequences:

- **Alternative-specific intercepts** $\alpha_j$ — each alternative gets its own intercept (a single common intercept is unidentifiable).
  - **Normalize one alternative's intercept to zero** as the baseline. The others measure the **difference vs. the baseline**. Any alternative can be the baseline; all choices of baseline are equivalent.
- **Individual-specific variables** (e.g., demographics like income — same value across alternatives) **must** have **alternative-specific coefficients**, with **one normalized to zero**. A common coefficient would create the same constant across alternatives and cancel out → unidentified.

### 2. Only the *scale* of utility matters (overall scale is irrelevant)

Multiplying all utilities by a common positive factor doesn't change the ranking → the scale isn't identified. We fix it by **normalizing the variance of the error term**.

> **Convenient:** the extreme value distribution already has a **fixed variance** ($\pi^2/6$), so in multinomial logit this scale normalization is **handled automatically** — nothing extra to do.

---

## Part 6 – Model Specification

### General specification

$$V_{ij} = \alpha_j + \beta x_{ij} + \gamma_j z_i + \delta_j w_{ij}$$

| Term | Variable type | Coefficient |
|---|---|---|
| $\alpha_j$ | — | **Alternative-specific intercepts** (one normalized to 0) |
| $\beta x_{ij}$ | Individual- **and** alternative-specific ($x_{ij}$) | **Generic** (common) coefficient |
| $\gamma_j z_i$ | Individual-specific only ($z_i$, e.g. demographics) | **Alternative-specific** coefficients (one normalized to 0) |
| $\delta_j w_{ij}$ | Individual- and alternative-specific ($w_{ij}$) | **Alternative-specific** coefficients |

> **Note:** A variable that is **only alternative-specific** (same for all individuals, e.g. a product's price as a pure constant per alternative) **cannot** be included alongside alternative-specific intercepts — it would be subsumed into the intercept and isn't separately identified.

### Basic specification (what we focus on in exercises)

$$V_{ij} = X_{ij}\beta_j = \beta_{0j} + \beta x_{ij}$$

- Only $x_{ij}$ variables that are **both** individual- and alternative-specific
- **Generic** (common) coefficients $\beta$ — same value across alternatives
- **Alternative-specific intercepts** $\beta_{0j}$, with **one normalized to zero**
- If there are $J$ alternatives, estimate only **$J - 1$ intercepts**

---

## Part 7 – Model Estimation (MLE)

Estimated by **Maximum Likelihood**, exactly as in binary logit. The likelihood is the product of each individual's chosen-alternative probability:

$$L(\beta) = \prod_{i=1}^{n} \prod_{j=1}^{J} (P_{ij})^{y_{ij}} = \prod_{i=1}^{n} \prod_{j=1}^{J} \left( \frac{e^{X_{ij}\beta_j}}{\sum_{k} e^{X_{ik}\beta_k}} \right)^{y_{ij}} = \prod_{i=1}^{n} \frac{e^{X_{ij_i}\beta_{j_i}}}{\sum_{k} e^{X_{ik}\beta_k}}$$

- $y_{ij} = 1$ if individual $i$ chooses alternative $j$, else $0$
- $j_i$ denotes the alternative individual $i$ actually chose
- Since $P_{ij}^{\,0} = 1$, only the **chosen** alternative's probability survives in the product

> **Highly favorable property:** the log-likelihood is **globally concave** in $\beta$, so the numerical search is guaranteed to converge and finds the optimum quickly — even with many variables and large data.

---

## Part 8 – Marginal Effects

Marginal effects measure how a one-unit change in an $x$ variable changes choice probabilities. In discrete choice they are more complex: the effect depends on **which alternative's $x$ changes** and **which alternative's probability is affected**. The full set forms a **$J \times J$ matrix**.

### Self-marginal effect (own probability)

Effect of alternative $j$'s own variable on alternative $j$'s probability:

$$\frac{\partial P_{ij}}{\partial x_{ij}} = \beta_{xj}\, P_{ij}(1 - P_{ij})$$

- **Same sign as $\beta$** — improving an alternative's attribute (positive $\beta$) raises its own choice probability. Intuitive.
- **Largest when $P_{ij} = \tfrac{1}{2}$**; smallest (≈ 0) when $P_{ij}$ is near **0 or 1**.

> Interpretation: if an alternative is already almost certain or almost never chosen, tweaking its attributes barely moves its probability. If it's a 50/50 call, improvements have the **biggest** payoff. Same spirit as the **S-shaped curve** from binary logit — steepest in the middle, flat at the ends.

### Cross-marginal effect (other alternatives' probability)

Effect of alternative $j$'s variable on a **different** alternative $j'$'s probability:

$$\frac{\partial P_{ij'}}{\partial x_{ij}} = -\beta_{xj}\, P_{ij}\, P_{ij'}$$

- **Opposite sign** to the self-effect — improving $j$ pulls probability **away** from competitors.
- $P_{ij}$ = probability of the alternative **whose $x$ is changing**; $P_{ij'}$ = probability of the alternative **being affected**.

### Effects sum to zero

$$\sum_{k=1}^{J} \frac{\partial P_{ik}}{\partial x_{ij}} = 0$$

> Because each individual always chooses **exactly one** alternative, any probability gained by the improved alternative is exactly offset by losses across the competitors — the net change is zero.

---

## Part 9 – R Implementation (`mlogit` package)

### Data formats: long vs. wide

The same information can be stored two ways. The `mlogit` package reads **both**.

| Format | Structure | Best when |
|---|---|---|
| **Long** | Each individual occupies **multiple rows** — one row per alternative | Natural for alternative-specific variables (e.g. cost per heating system) |
| **Wide** | Each individual occupies **one row**; create a separate column per alternative (e.g. `ic.GC`, `ic.GR`, `ic.EC`, `ic.ER`, `ic.HP`) | Efficient when you have many individual-specific (alternative-invariant) variables, since they appear **once** instead of repeated |

> In the **heating data** demo: 900 newly built single-family California homes with central AC. The choice is the heating system among **5 alternatives** — gas central (GC), gas room (GR), electric central (EC), electric room (ER), heat pump (HP). Variables: installation cost `ic`, annual operating cost `oc` (both alternative-specific), choice indicator `depvar`, alternative name `alt`, choice ID `chid`. We use the **long format** since there are no individual-specific-only variables.

### Setup and data import

```r
# install.packages("mlogit")   # only once per computer
library(mlogit)

mydata <- read.csv("heating_data.csv")   # header = TRUE by default
str(mydata)
head(mydata)
```

### Convert to an mlogit data object

```r
mldata <- mlogit.data(
  mydata,
  shape    = "long",     # "long" or "wide"
  choice   = "depvar",   # variable holding the choice decision (1 = chosen)
  alt.var  = "alt",      # variable holding the alternative names
  chid.var = "chid"      # variable holding the choice/individual ID
)
head(mldata, 10)   # first 10 rows = first 2 houses (5 rows each)
```

### Estimate the multinomial logit model

```r
# Basic spec: generic coefficients on ic and oc, plus alternative-specific intercepts
ml_model <- mlogit(depvar ~ ic + oc, data = mldata)
summary(ml_model)

# For the full/general spec, separate variable groups with the pipe |:
#   mlogit(depvar ~ x_alt_generic | z_individual | w_alt_specific, data = mldata)
```

### Interpreting the coefficient table

```r
# The first J-1 = 4 rows are ALTERNATIVE-SPECIFIC INTERCEPTS.
#   - 5 alternatives, but only 4 intercepts; EC (electric central) is the baseline (normalized to 0).
#   - Positive intercept  -> that option gives higher avg utility than baseline.
#   - Negative intercept  -> lower avg utility than baseline.
# ic and oc each get ONE generic coefficient (both individual- and alternative-specific variables).
#   - Both are NEGATIVE: higher install/operating cost -> lower utility -> makes sense.
```

### Predicted choice probabilities

```r
# fitted() returns choice probabilities. MUST set outcome = FALSE to get ALL alternatives;
# the default outcome = TRUE returns only the chosen alternative's probability.
probs <- fitted(ml_model, outcome = FALSE)
head(probs)   # rows sum to 1; gas central (GC) generally has the highest probability
```

### Marginal effects (the J×J matrix)

```r
# Evaluate at the MEAN of each x variable, computed PER ALTERNATIVE.
# In long format you CANNOT take a plain column mean — use tapply() to average BY alternative.
ic_means <- tapply(mydata$ic, mydata$alt, mean)   # 5 alternative-specific means
oc_means <- tapply(mydata$oc, mydata$alt, mean)   # 5 alternative-specific means

# Store the evaluation point in a data frame (what effects() expects)
z <- data.frame(ic = ic_means, oc = oc_means)

# Marginal effects of a chosen covariate (here, operating cost oc)
me <- effects(ml_model, covariate = "oc", data = z)
me   # a 5x5 (J x J) matrix
```

**Reading the matrix:**
- **Diagonal cells = self-marginal effects** (own attribute → own probability), **negative** for cost (higher cost → lower own probability).
- **Off-diagonal cells = cross-marginal effects** (one alternative's attribute → another's probability), **positive** for cost (a competitor getting more expensive raises this option's probability).
- Example: row EC, col EC ≈ **−0.044%** (raising EC's operating cost lowers EC's own probability). Row GC, col GR ≈ **+0.0647%** (raising GC's operating cost raises GR's probability).

> **Symmetry note:** the matrix is **symmetric only because we used generic (common) $\beta$** coefficients. With **alternative-specific** $\beta$, the effect of $j$ on $j'$ need not equal the effect of $j'$ on $j$, so the matrix is **not necessarily symmetric**.

> **Key R functions summary:**
> - `library(mlogit)` → load the multinomial logit package
> - `mlogit.data(..., shape, choice, alt.var, chid.var)` → build the choice data object
> - `mlogit(depvar ~ vars, data = ...)` → estimate the model (use `|` for general specs)
> - `fitted(model, outcome = FALSE)` → choice probabilities for **all** alternatives
> - `tapply(x, alt, mean)` → alternative-specific means in **long** format
> - `effects(model, covariate, data)` → the $J \times J$ marginal-effects matrix

---

## Summary

- **Discrete choice** generalizes binary response to a **finite set of alternatives** that are mutually exclusive, exhaustive, and finite.
- Built on the **Random Utility Model**: $U_{ij} = V_{ij} + \varepsilon_{ij}$, individuals pick the highest-utility alternative.
- **Multinomial logit** assumes i.i.d. **extreme value** errors → the clean closed form $P_{ij} = e^{V_{ij}} / \sum_k e^{V_{ik}}$.
- **Binary logit is a special case** (2 alternatives, one utility normalized to 0; error difference is logistic).
- **Identification:** only **differences** and **scale** of utility matter → alternative-specific intercepts with one normalized to 0; individual-specific variables need alternative-specific coefficients; scale is auto-normalized by the EV variance.
- **Estimation** is by **MLE** with a **globally concave** log-likelihood.
- **Marginal effects** are a **$J \times J$ matrix**: self-effects $\beta P_{ij}(1-P_{ij})$ (max at $P=\tfrac12$), cross-effects $-\beta P_{ij}P_{ij'}$, and all effects of one variable **sum to zero**.

---

## Key Formulas Cheat Sheet

| Quantity | Formula |
|---|---|
| Random utility | $U_{ij} = V_{ij} + \varepsilon_{ij}$ |
| EV (Gumbel) PDF / CDF | $f(\varepsilon)=e^{-\varepsilon}e^{-e^{-\varepsilon}}$, $\ F(\varepsilon)=e^{-e^{-\varepsilon}}$ |
| EV mean / variance | $E[\varepsilon]=\gamma\approx0.5772$, $\ \text{Var}=\pi^2/6$ |
| Choice probability | $P_{ij} = \dfrac{e^{V_{ij}}}{\sum_k e^{V_{ik}}}$ |
| Binary logit (special case) | $\Pr(y_i=1)=\dfrac{e^{V}}{1+e^{V}}$ |
| Basic specification | $V_{ij}=\beta_{0j}+\beta x_{ij}$ (one $\beta_{0j}=0$; $J-1$ intercepts) |
| Likelihood | $L(\beta)=\prod_i \dfrac{e^{X_{ij_i}\beta_{j_i}}}{\sum_k e^{X_{ik}\beta_k}}$ |
| Self-marginal effect | $\dfrac{\partial P_{ij}}{\partial x_{ij}} = \beta_{xj}P_{ij}(1-P_{ij})$ |
| Cross-marginal effect | $\dfrac{\partial P_{ij'}}{\partial x_{ij}} = -\beta_{xj}P_{ij}P_{ij'}$ |
| Effects sum to zero | $\sum_{k=1}^{J} \dfrac{\partial P_{ik}}{\partial x_{ij}} = 0$ |
