---
title: "Introduction to Causality"
layout: single
permalink: /causality/
author_profile: true
toc: true
toc_label: "Table of Contents"
toc_icon: "book"
---

Causality is one of the most fundamental concepts in science, philosophy, and artificial intelligence. It is concerned with understanding **why** things happen, not just **what** happens. This distinction separates causal reasoning from purely statistical or correlational analysis.

---

## What is Causality?

The classical statistical paradigm operates at the level of associations. It can tell us that two variables tend to co-occur, but it cannot tell us whether one *produces* the other. Causality fills this gap by introducing a formal framework for reasoning about **cause-and-effect relationships**.

Judea Pearl introduced the **Ladder of Causation** — a three-level hierarchy organizing causal reasoning by increasing depth:

| Level | Name | Question | Notation |
|-------|------|----------|----------|
| 1 | Association (Seeing) | What patterns do we observe? | P(Y \| X) |
| 2 | Intervention (Doing) | What happens if we actively change X? | P(Y \| do(X)) |
| 3 | Counterfactual (Imagining) | What would Y have been had X been different? | P(Y_x' \| X = x) |

Standard machine learning operates almost entirely at Level 1. Causal inference aspires to reach all three levels.

---

## Directed Acyclic Graphs (DAGs)

### Definition

A **Directed Acyclic Graph (DAG)** is a graphical representation of a causal structure. A DAG `G = (V, E)` consists of:

- A set of **nodes** `V = {X_1, X_2, ..., X_n}`, each representing a random variable.
- A set of **directed edges** `E`, where an edge `X_i → X_j` means `X_i` is a **direct cause** of `X_j`.
- The **acyclicity** constraint: no variable can be its own ancestor.

---

### Structural Motifs

Every DAG is built from three elementary structures. Understanding them is essential for reading causal graphs.

---

#### 1. Chain (Mediation)

```
X ──→ Z ──→ Y
```

`Z` **mediates** the effect of `X` on `Y`. The path `X → Z → Y` is **open** by default. Conditioning on `Z` **blocks** this path.

**Example:** Smoking `(X)` causes tar buildup `(Z)`, which causes cancer `(Y)`.

---

#### 2. Fork (Common Cause / Confounder)

```
X ←── Z ──→ Y
```

`Z` is a **common cause** of `X` and `Y`, inducing a spurious association between them. The path is **open** by default. Conditioning on `Z` **blocks** this path and removes the confounding.

**Example:** A gene `(Z)` causes both smoking behavior `(X)` and cancer `(Y)`. Without adjusting for `Z`, we may incorrectly attribute `Y` to `X`.

---

#### 3. Collider

```
X ──→ Z ←── Y
```

`Z` is a **common effect** of `X` and `Y`. This path is **blocked** by default. Conditioning on `Z` **opens** the path — this is known as **collider bias** or **Berkson's paradox**.

**Example:** Both talent `(X)` and luck `(Y)` cause success `(Z)`. Among successful people (conditioning on `Z`), talent and luck appear negatively correlated — even if they are independent in the full population.

---

### Summary of Path Rules

| Structure | Default State | After Conditioning on Middle Node |
|-----------|--------------|----------------------------------|
| Chain: `X → Z → Y` | Open | Blocked |
| Fork: `X ← Z → Y` | Open | Blocked |
| Collider: `X → Z ← Y` | Blocked | Opened |

---

### d-Separation

**d-separation** is the graphical criterion for reading conditional independencies from a DAG. Two sets of variables `X` and `Y` are **d-separated** by a set `Z` if every path between them is blocked by `Z`.

If `X` and `Y` are d-separated by `Z`, then in any distribution **faithful** to `G`:

```
X  ⊥⊥  Y  |  Z
(X is conditionally independent of Y given Z)
```

---

### Markov Factorization

The **Markov condition** allows the joint distribution to be factored as a product of local conditionals:

```
P(X_1, X_2, ..., X_n)  =  ∏ P(X_i | Pa(X_i))
                            i
```

where `Pa(X_i)` denotes the **parents** (direct causes) of `X_i` in the DAG.

**Example DAG:**

```
     Z
    ↙ ↘
   X → Y
```

Markov factorization gives:

```
P(Z, X, Y)  =  P(Z) · P(X | Z) · P(Y | X, Z)
```

---

## Structural Causal Models (SCMs)

### Definition

An **SCM** `M = <U, V, F, P(U)>` consists of:

- `V = {X_1, ..., X_n}` — **endogenous variables** (modeled explicitly).
- `U = {U_1, ..., U_n}` — **exogenous variables** (background noise, unmodeled causes).
- `F = {f_1, ..., f_n}` — **structural equations**, one per endogenous variable:

```
X_i  :=  f_i(Pa(X_i), U_i)
```

- `P(U)` — a joint distribution over the exogenous variables.

The `:=` symbol denotes **assignment** (not equality) — it captures the asymmetric nature of causation.

---

### SCM Example 1: Simple Treatment and Outcome

**DAG:**

```
U_T        U_Y
 |           |
 ↓           ↓
 T ─────────→ Y
```

**Structural equations:**

```
T  :=  f_T(U_T)
Y  :=  f_Y(T, U_Y)
```

If `U_T ⊥⊥ U_Y` (no hidden confounding), the model is **Markovian** and all causal effects are identifiable from observational data.

**Joint distribution:**

```
P(T, Y)  =  P(T) · P(Y | T)
```

---

### SCM Example 2: Observed Confounder

**DAG:**

```
      Z
     ↙ ↘
    T → Y
```

**Structural equations:**

```
Z  :=  f_Z(U_Z)
T  :=  f_T(Z, U_T)
Y  :=  f_Y(T, Z, U_Y)
```

**Joint distribution:**

```
P(Z, T, Y)  =  P(Z) · P(T | Z) · P(Y | T, Z)
```

`Z` is a measured confounder. The backdoor adjustment on `Z` recovers the causal effect of `T` on `Y`.

---

### SCM Example 3: Hidden Confounder

**DAG:**

```
U (hidden)
 ↙          ↘
 T ─────────→ Y
```

Represented with a **bidirected arc** (dashed) to indicate hidden common cause:

```
T <----> Y    (hidden confounder U)
T ──────→ Y   (direct causal effect)
```

Combined:

```
T ⇠⇢ Y   and   T ──→ Y
```

**Structural equations:**

```
U  :=  f_U(U_0)
T  :=  f_T(U, U_T)
Y  :=  f_Y(T, U, U_Y)
```

Because `U` is unobserved, simple regression of `Y` on `T` is **biased**. We cannot use the backdoor formula — we need either a mediator (frontdoor) or an instrument.

---

### SCM Example 4: Mediation

**DAG:**

```
X ──→ M ──→ Y
 \          ↑
  ──────────
   (direct)
```

**Structural equations:**

```
X  :=  f_X(U_X)
M  :=  f_M(X, U_M)
Y  :=  f_Y(X, M, U_Y)
```

**Joint distribution:**

```
P(X, M, Y)  =  P(X) · P(M | X) · P(Y | X, M)
```

The total effect of `X` on `Y` decomposes into:

```
Total Effect  =  Direct Effect (X → Y)
               + Indirect Effect (X → M → Y)
```

---

## Interventions and the do-Calculus

### The do-Operator

An intervention `do(X = x)` represents an external manipulation that **forces** `X` to take value `x`, cutting all incoming edges to `X` in the DAG. This produces a **mutilated graph** `G_{\bar{X}}`:

```
Original graph:        Mutilated graph after do(X = x):

  Z                        Z
  ↓                        
  X → Y                X = x → Y    (edge Z→X removed)
```

The **interventional distribution** `P(Y | do(X = x))` is the distribution of `Y` in this mutilated model.

---

### Conditioning vs. Intervention

| Operation | Notation | Meaning |
|-----------|----------|---------|
| Conditioning | `P(Y \| X = x)` | Restrict to units where X happened to be x |
| Intervention | `P(Y \| do(X = x))` | Force X to be x; remove all causes of X |

These are **not** the same in the presence of confounding.

**Example:** In the forked DAG `X ← Z → Y`:

```
P(Y | X = x)      ≠      P(Y | do(X = x))
```

The first mixes the causal effect with the confounding influence of `Z`. The second isolates only the causal effect.

---

### The Backdoor Criterion

A set `Z` satisfies the **backdoor criterion** relative to `(X, Y)` if:

1. No node in `Z` is a **descendant** of `X`.
2. `Z` **blocks** every **backdoor path** from `X` to `Y` (paths entering `X` via an arrow pointing into `X`).

If `Z` satisfies the backdoor criterion:

```
P(Y | do(X = x))  =  Σ_z  P(Y | X=x, Z=z) · P(Z=z)
```

---

### Case 1: Single Observed Confounder

**DAG:**

```
    Z
   ↙ ↘
   X → Y
```

**Backdoor path:** `X ← Z → Y`

`Z` is not a descendant of `X`, and conditioning on `Z` blocks the backdoor path. The backdoor criterion is satisfied.

**Adjustment formula:**

```
P(Y | do(X = x))  =  Σ_z  P(Y | X=x, Z=z) · P(Z=z)
```

**Derivation:**

```
P(Y | do(X = x))
  = Σ_z  P(Y | do(X=x), Z=z) · P(Z=z | do(X=x))      [total probability]
  = Σ_z  P(Y | do(X=x), Z=z) · P(Z=z)                 [do(X) removes Z→X, so Z ⊥ do(X)]
  = Σ_z  P(Y | X=x, Z=z)     · P(Z=z)                 [Z blocks all backdoor paths]
```

The last step uses the fact that once `Z` is conditioned on, the remaining path from `X` to `Y` is purely causal — so `do(X=x)` is equivalent to observing `X=x`.

---

### Case 2: Two Independent Confounders

**DAG:**

```
  Z1    Z2
  ↓  ↘↙  ↓
  X ──────→ Y
```

**Backdoor paths:**
```
X ← Z1 → Y
X ← Z2 → Y
```

Both paths are blocked by conditioning on `Z = {Z1, Z2}`.

**Adjustment formula:**

```
P(Y | do(X = x))  =  Σ_{z1,z2}  P(Y | X=x, Z1=z1, Z2=z2) · P(Z1=z1, Z2=z2)
```

If `Z1 ⊥⊥ Z2` (independent confounders):

```
P(Y | do(X = x))  =  Σ_{z1,z2}  P(Y | X=x, Z1=z1, Z2=z2) · P(Z1=z1) · P(Z2=z2)
```

---

### Case 3: Mediator — Do NOT Adjust

**DAG:**

```
X ──→ M ──→ Y
```

`M` is a mediator. There are **no backdoor paths** from `X` to `Y` — the only path is the causal path itself.

```
P(Y | do(X = x))  =  P(Y | X = x)    [no adjustment needed]
```

**Warning:** Conditioning on `M` would **block** the indirect path `X → M → Y` and give only the direct effect — which is a different quantity and likely not what is intended.

---

### Case 4: Collider on Backdoor Path — Do NOT Adjust

**DAG:**

```
X ──→ C ←── Z ──→ Y
             ↑
             X (also causes Z? No — see below)
```

A cleaner example:

```
W ──→ X
W ──→ C
V ──→ C
V ──→ Y
```

Here `C` is a **collider** on the path `X ← W → C ← V → Y`. This path is **blocked** by default.

```
Do NOT condition on C.
```

If you condition on `C`, the path `X ← W → C ← V → Y` opens, introducing spurious association between `X` and `Y` through the backdoor.

**Rule:** Never include a **collider** (or its descendants) in the adjustment set on any backdoor path.

---

### Case 5: Frontdoor Adjustment (Hidden Confounder)

**DAG:**

```
  U (hidden)
 ↙            ↘
 X ────→ M ────→ Y
```

There is a hidden confounder `U` between `X` and `Y`. We **cannot** use the backdoor formula because `U` is unobserved. However, `M` (a mediator) satisfies the **frontdoor criterion**:

1. `M` intercepts all directed paths from `X` to `Y`.
2. There are no unblocked backdoor paths from `X` to `M`.
3. All backdoor paths from `M` to `Y` are blocked by `X`.

**Frontdoor adjustment formula:**

```
P(Y | do(X = x))  =  Σ_m  P(M=m | X=x) · Σ_{x'}  P(Y | X=x', M=m) · P(X=x')
```

**Derivation:**

**Step 1** — Effect of `X` on `M`:

No backdoor paths from `X` to `M` (the path `X ← U → Y ← M` would require `Y` to be a collider ancestor — there is no such path). So:

```
P(M=m | do(X=x))  =  P(M=m | X=x)
```

**Step 2** — Effect of `M` on `Y`:

The backdoor path `M ← X ← U → Y` is blocked by conditioning on `X`:

```
P(Y | do(M=m))  =  Σ_{x'}  P(Y | X=x', M=m) · P(X=x')
```

**Step 3** — Chain the two steps:

```
P(Y | do(X=x))
  =  Σ_m  P(M=m | do(X=x))  ·  P(Y | do(M=m))
  =  Σ_m  P(M=m | X=x)  ·  Σ_{x'}  P(Y | X=x', M=m) · P(X=x')
```

This is remarkable: even with a **hidden confounder** between `X` and `Y`, the causal effect is identifiable through the mediator `M` using only observational data.

---

### The do-Calculus: Three Rules

When no valid adjustment set exists, the **do-calculus** provides three inference rules for deriving interventional distributions from observational ones.

Let `G_{\bar{X}}` = DAG with all edges **into** `X` removed.
Let `G_{\underline{X}}` = DAG with all edges **out of** `X` removed.

**Rule 1 — Insertion/Deletion of Observations:**

```
If  (Y ⊥⊥ Z | X, W)  in  G_{\bar{X}}  :

    P(Y | do(X), Z, W)  =  P(Y | do(X), W)
```

**Rule 2 — Action/Observation Exchange:**

```
If  (Y ⊥⊥ Z | X, W)  in  G_{\bar{X}, \underline{Z}}  :

    P(Y | do(X), do(Z), W)  =  P(Y | do(X), Z, W)
```

**Rule 3 — Insertion/Deletion of Actions:**

```
If  (Y ⊥⊥ Z | X, W)  in  G_{\bar{X}, \bar{Z(W)}}  :

    P(Y | do(X), do(Z), W)  =  P(Y | do(X), W)
```

Pearl (2000) proved the do-calculus is **complete**: if a causal quantity cannot be identified by these rules, it is not identifiable from observational data given the assumed graph.

---

## Counterfactuals

### Definition

A counterfactual asks:

> *"What would Y have been, had X been x', given that we observed X = x and Y = y?"*

Notation: `Y_{x'}` denotes the **potential outcome** of `Y` under the intervention `do(X = x')`.

Counterfactuals operate at the **individual level** — they reason about what *would have happened* in a specific situation, contrary to what actually occurred.

---

### The 3-Step Algorithm

Given an SCM `M` and evidence `E = e`:

**Step 1 — Abduction:**

```
Update:  P(U)  →  P(U | E = e)

Use Bayes' theorem to infer the background conditions
that generated the observed data.
```

**Step 2 — Action:**

```
Mutilate the model:  M  →  M_{x'}

Replace equation  X := f_X(Pa(X), U_X)
            with  X := x'
Remove all edges into X.
```

**Step 3 — Prediction:**

```
Compute:  P(Y_{x'} = y | E = e)

Use the mutilated model M_{x'} under the updated noise P(U | E = e).
```

---

### Worked Example: Drug Treatment

**SCM:**

```
U_X              U_Y
 |                |
 ↓                ↓
 X ──────────────→ Y

X  :=  U_X               (U_X ~ Bernoulli(0.5))
Y  :=  X + U_Y            (U_Y ~ Normal(0, 1))
```

**Observation:** `X = 1, Y = 2.3`. What would `Y` have been had `X = 0`?

**Step 1 — Abduction:**

```
Y = X + U_Y
2.3 = 1 + U_Y
U_Y = 1.3

So:  P(U_Y | X=1, Y=2.3)  is a point mass at  U_Y = 1.3
```

**Step 2 — Action:**

```
Mutilate:  X := 0    (force X to 0)
```

**Step 3 — Prediction:**

```
Y_{X=0}  =  0 + U_Y  =  0 + 1.3  =  1.3
```

**Answer:** *"Had the patient not received the drug, their outcome would have been 1.3 instead of 2.3."*

---

### Individual Treatment Effects (ITE)

The **potential outcome** framework defines:

```
Y_i(1)  =  outcome of unit i under treatment
Y_i(0)  =  outcome of unit i under control
```

The **Individual Treatment Effect (ITE):**

```
ITE_i  =  Y_i(1) - Y_i(0)
```

**Fundamental Problem of Causal Inference** (Holland, 1986): We can never observe both `Y_i(1)` and `Y_i(0)` for the same unit simultaneously — one is always the counterfactual.

Population-level estimands:

```
ATE   =  E[ Y(1) - Y(0) ]                       (Average Treatment Effect)
ATT   =  E[ Y(1) - Y(0)  |  T = 1 ]             (Average Treatment Effect on Treated)
CATE  =  E[ Y(1) - Y(0)  |  X = x ]             (Conditional Average Treatment Effect)
```

---

### Counterfactual Fairness

A decision `Ŷ` is **counterfactually fair** with respect to a protected attribute `A` (Kusner et al., 2017) if:

```
P(Ŷ_{A←a} = y | X=x, A=a)  =  P(Ŷ_{A←a'} = y | X=x, A=a)
```

for all `y`, all values `a, a'` of the protected attribute, and all individuals `x`.

In plain language: *the decision would remain the same even if the individual's protected characteristic had been different*, holding all causally independent background factors fixed.

This has direct implications for **Explainable AI** and **trustworthy machine learning**.

---

## Summary

| Concept | Key Idea |
|---------|----------|
| DAG | Graphical encoding of causal assumptions |
| d-Separation | Graphical criterion for conditional independence |
| SCM | Structural equations + noise variables |
| do-operator | Formal model of intervention (cuts incoming edges) |
| Backdoor criterion | Sufficient condition for adjustment with observed confounders |
| Frontdoor criterion | Identifies causal effect through mediator when confounder is hidden |
| do-Calculus | Complete set of rules for deriving interventional distributions |
| Counterfactual | Individual-level reasoning about alternative worlds |
| ITE / ATE | Measures of causal effect at individual and population level |
| Counterfactual Fairness | Decision invariance under protected attribute changes |

---

## Further Reading

- Pearl, J. (2009). *Causality: Models, Reasoning, and Inference* (2nd ed.). Cambridge University Press.
- Pearl, J., Glymour, M., & Jewell, N. P. (2016). *Causal Inference in Statistics: A Primer*. Wiley.
- Peters, J., Janzing, D., & Schölkopf, B. (2017). *Elements of Causal Inference*. MIT Press.
- Hernán, M. A., & Robins, J. M. (2020). *Causal Inference: What If*. Chapman & Hall/CRC.
- Kusner, M. J., Loftus, J., Russell, C., & Silva, R. (2017). Counterfactual Fairness. *NeurIPS*.
