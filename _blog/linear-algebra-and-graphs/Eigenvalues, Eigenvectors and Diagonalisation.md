---
title: "Eigenvalues, Eigenvectors and Diagonalisation"
layout: single
author_profile: true
mathjax: true
---

<style>
.def-box {
  background: #f0f7ff;
  border-left: 4px solid #2a7ae2;
  padding: 1em 1.2em;
  margin: 1.2em 0;
  border-radius: 0 6px 6px 0;
}
.thm-box {
  background: #fff8f0;
  border-left: 4px solid #e2852a;
  padding: 1em 1.2em;
  margin: 1.2em 0;
  border-radius: 0 6px 6px 0;
}
.example-box {
  background: #f0fff4;
  border-left: 4px solid #2ae27a;
  padding: 1em 1.2em;
  margin: 1.2em 0;
  border-radius: 0 6px 6px 0;
}
.exercise-box {
  background: #fdf0ff;
  border-left: 4px solid #a02ae2;
  padding: 1em 1.2em;
  margin: 1.2em 0;
  border-radius: 0 6px 6px 0;
}
.solution-box {
  background: #f9f9f9;
  border-left: 4px solid #888;
  padding: 1em 1.2em;
  margin: 1.2em 0;
  border-radius: 0 6px 6px 0;
}
.intuition-box {
  background: #fffde7;
  border-left: 4px solid #f0c420;
  padding: 1em 1.2em;
  margin: 1.2em 0;
  border-radius: 0 6px 6px 0;
}
.box-label {
  font-weight: bold;
  font-size: 0.95em;
  margin-bottom: 0.4em;
  display: block;
}
.step-table {
  width: 100%;
  border-collapse: collapse;
  margin: 1em 0;
}
.step-table th {
  background: #2a7ae2;
  color: white;
  padding: 0.5em 1em;
  text-align: left;
}
.step-table td {
  padding: 0.5em 1em;
  border-bottom: 1px solid #ddd;
}
.step-table tr:nth-child(even) { background: #f5f5f5; }
</style>

## 1. Eigenvalues and Eigenvectors

### Definitions

<div class="def-box">
<span class="box-label">Definition 1 — Eigenvalue and Eigenvector</span>
Let $\mathbf{A} \in \mathbb{R}^{n\times n}$. A nonzero vector $\mathbf{v} \in \mathbb{R}^n$ is called an <strong>eigenvector</strong> of $\mathbf{A}$ if there exists a scalar $\lambda \in \mathbb{C}$ such that

$$\mathbf{A}\mathbf{v} = \lambda\mathbf{v}.$$

The scalar $\lambda$ is called the corresponding <strong>eigenvalue</strong>.
The set of all eigenvalues of $\mathbf{A}$ is called the <strong>spectrum</strong> of $\mathbf{A}$, denoted $\sigma(\mathbf{A})$.
</div>

### Geometric Intuition: What Do Eigenvalues Actually Mean?

<div class="intuition-box">
<span class="box-label">💡 Geometric Intuition</span>

Think of $\mathbf{A}$ as a machine that takes a vector and transforms it — rotating it, stretching it, shearing it, or flipping it.

For **most** vectors $\mathbf{x}$, after applying $\mathbf{A}$, the result $\mathbf{A}\mathbf{x}$ points in a completely different direction and has a different length. Both direction and magnitude change simultaneously.

But there exist **special** vectors — eigenvectors — for which something remarkable happens: after applying $\mathbf{A}$, the vector **still points in the same direction** (or exactly opposite). Only its length changes, by the factor $|\lambda|$.

The eigenvalue $\lambda$ tells you exactly what happens to those special directions:

- $\lambda > 1$: the eigenvector is **stretched** (grows longer).
- $0 < \lambda < 1$: the eigenvector is **compressed** (shrinks).
- $\lambda = 1$: the eigenvector is **left completely unchanged**.
- $\lambda < 0$: the eigenvector is **flipped** (reversed direction) and possibly scaled.
- $\lambda = 0$: the eigenvector is **collapsed to zero** — it lands in the null space of $\mathbf{A}$.

**Key insight.** Eigenvectors are the "skeleton" of a linear transformation. Once you know them, you understand the fundamental structure of how $\mathbf{A}$ acts on all of space.
</div>

### Finding Eigenvalues: The Characteristic Polynomial

From $\mathbf{A}\mathbf{v} = \lambda\mathbf{v}$, rearranging gives:

$$(\mathbf{A} - \lambda\mathbf{I})\mathbf{v} = \mathbf{0}.$$

For this to have a nontrivial solution ($\mathbf{v} \neq \mathbf{0}$), the matrix $(\mathbf{A} - \lambda\mathbf{I})$ must be **singular** (non-invertible):

$$\boxed{\det(\mathbf{A} - \lambda\mathbf{I}) = 0.}$$

This is the **characteristic equation**. The polynomial $p(\lambda) = \det(\mathbf{A}-\lambda\mathbf{I})$ of degree $n$ is the **characteristic polynomial**.

<div class="thm-box">
<span class="box-label">Theorem 2 — Fundamental Theorem Applied to Eigenvalues</span>
Every $n \times n$ matrix has exactly $n$ eigenvalues (counting algebraic multiplicity) in $\mathbb{C}$. Over $\mathbb{R}$, complex eigenvalues always appear in conjugate pairs.
</div>

<div class="def-box">
<span class="box-label">Definition 3 — Eigenspace</span>
Given eigenvalue $\lambda$, the <strong>eigenspace</strong> is

$$E_\lambda = \mathrm{Null}(\mathbf{A} - \lambda\mathbf{I}) = \{\mathbf{v} \in \mathbb{R}^n : \mathbf{A}\mathbf{v} = \lambda\mathbf{v}\}.$$

It is a subspace of $\mathbb{R}^n$. Its dimension is the <strong>geometric multiplicity</strong> of $\lambda$.
The <strong>algebraic multiplicity</strong> is the multiplicity of $\lambda$ as a root of $p(\lambda)$. Always:

$$\text{geometric multiplicity}(\lambda) \leq \text{algebraic multiplicity}(\lambda).$$
</div>

<div class="intuition-box">
<span class="box-label">💡 Geometric Intuition — The Eigenspace as a Stable Subspace</span>

The eigenspace $E_\lambda$ is the collection of all eigenvectors for a given $\lambda$ (plus the zero vector). It forms a subspace — a line, a plane, or higher-dimensional analog — that $\mathbf{A}$ maps **entirely into itself**. Every vector inside $E_\lambda$ is simply scaled by $\lambda$; no vector escapes the subspace. Think of it as a "stable region" of the transformation.
</div>

### Worked Examples

<div class="example-box">
<span class="box-label">Example 4 — 2×2 Matrix</span>

Find the eigenvalues and eigenvectors of $\mathbf{A} = \begin{pmatrix} 3 & 1 \\ 0 & 2 \end{pmatrix}.$

**Step 1: Characteristic polynomial.**

$$p(\lambda) = \det\begin{pmatrix} 3-\lambda & 1 \\ 0 & 2-\lambda \end{pmatrix} = (3-\lambda)(2-\lambda) = \lambda^2 - 5\lambda + 6.$$

Roots: $\lambda_1 = 3$, $\lambda_2 = 2$.

**Step 2: Eigenvectors.**

For $\lambda_1 = 3$:

$$(\mathbf{A}-3\mathbf{I})\mathbf{v} = \mathbf{0} \implies \begin{pmatrix} 0 & 1 \\ 0 & -1 \end{pmatrix}\mathbf{v} = \mathbf{0} \implies v_2 = 0 \implies \mathbf{v}_1 = \begin{pmatrix}1\\0\end{pmatrix}.$$

For $\lambda_2 = 2$:

$$(\mathbf{A}-2\mathbf{I})\mathbf{v} = \mathbf{0} \implies \begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}\mathbf{v} = \mathbf{0} \implies v_1 = -v_2 \implies \mathbf{v}_2 = \begin{pmatrix}-1\\1\end{pmatrix}.$$

**Geometric meaning:** $\mathbf{v}_1 = (1,0)^T$ (the $x$-axis) is stretched by factor 3 — it triples in length but stays horizontal. $\mathbf{v}_2 = (-1,1)^T$ is stretched by factor 2 — it doubles in length but keeps its direction. Every other vector gets both rotated and scaled; only these two directions are preserved.
</div>

<div class="example-box">
<span class="box-label">Example 5 — Symmetric Matrix</span>

Find eigenvalues and eigenvectors of $\mathbf{B} = \begin{pmatrix} 4 & 2 \\ 2 & 1 \end{pmatrix}.$

**Characteristic polynomial:**

$$p(\lambda) = (4-\lambda)(1-\lambda) - 4 = \lambda^2 - 5\lambda = \lambda(\lambda-5).$$

So $\lambda_1 = 0$, $\lambda_2 = 5$.

For $\lambda_1 = 0$: $\quad \mathbf{v}_1 = \begin{pmatrix}1\\-2\end{pmatrix}.$

For $\lambda_2 = 5$: $\quad \mathbf{v}_2 = \begin{pmatrix}2\\1\end{pmatrix}.$

Note: $\mathbf{v}_1 \cdot \mathbf{v}_2 = (1)(2)+(-2)(1) = 0$. The eigenvectors are **orthogonal** — this always holds for symmetric matrices (the Spectral Theorem).
</div>

### Exercises

<div class="exercise-box">
<span class="box-label">Exercise 6 — Computing Eigenvalues and Eigenvectors</span>

**(a)** Find all eigenvalues and eigenvectors of $\mathbf{A} = \begin{pmatrix}2&3\\0&5\end{pmatrix}$.

**(b)** Find all eigenvalues and eigenvectors of $\mathbf{B} = \begin{pmatrix}1&1\\1&1\end{pmatrix}$.

**(c)** Show that if $\lambda$ is an eigenvalue of $\mathbf{A}$, then $\lambda^2$ is an eigenvalue of $\mathbf{A}^2$.

**(d)** A matrix $\mathbf{P}$ satisfies $\mathbf{P}^2 = \mathbf{P}$ (a *projection*). What are the only possible eigenvalues of $\mathbf{P}$? Give a geometric reason.
</div>

<div class="solution-box">
<span class="box-label">Solutions</span>

**(a)** Characteristic polynomial: $(2-\lambda)(5-\lambda)=0$, so $\lambda_1=2$, $\lambda_2=5$.
For $\lambda_1=2$: $\mathbf{v}_1=(1,0)^T$. For $\lambda_2=5$: $\mathbf{v}_2=(1,1)^T$.

**(b)** Characteristic polynomial: $\lambda(\lambda-2)=0$, so $\lambda_1=0$, $\lambda_2=2$.
For $\lambda_1=0$: $\mathbf{v}_1=(1,-1)^T$. For $\lambda_2=2$: $\mathbf{v}_2=(1,1)^T$.

**(c)** If $\mathbf{A}\mathbf{v}=\lambda\mathbf{v}$, then $\mathbf{A}^2\mathbf{v}=\mathbf{A}(\lambda\mathbf{v})=\lambda^2\mathbf{v}$. ✓

**(d)** From $\mathbf{P}^2=\mathbf{P}$ and $\mathbf{P}\mathbf{v}=\lambda\mathbf{v}$, we get $\lambda^2=\lambda$, so $\lambda \in \{0,1\}$. Geometrically: a projection either collapses a vector to zero (eigenvalue 0) or leaves it fixed in the projection subspace (eigenvalue 1).
</div>

---

## 2. Diagonalisation

### Definition and Construction

<div class="def-box">
<span class="box-label">Definition 7 — Diagonalisable Matrix</span>
A matrix $\mathbf{A} \in \mathbb{R}^{n\times n}$ is <strong>diagonalisable</strong> if there exists an invertible matrix $\mathbf{P}$ and a diagonal matrix $\mathbf{D}$ such that

$$\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}.$$

Equivalently, $\mathbf{A}$ is diagonalisable if and only if it has $n$ linearly independent eigenvectors.
</div>

**How to build $\mathbf{P}$ and $\mathbf{D}$.** Place each eigenvector as a column of $\mathbf{P}$, and the corresponding eigenvalue on the diagonal of $\mathbf{D}$:

$$\mathbf{P} = \begin{pmatrix} | & | & & | \\ \mathbf{v}_1 & \mathbf{v}_2 & \cdots & \mathbf{v}_n \\ | & | & & | \end{pmatrix}, \qquad \mathbf{D} = \begin{pmatrix} \lambda_1 & & \\ & \ddots & \\ & & \lambda_n \end{pmatrix}.$$

### Geometric Intuition: What Is Diagonalisation Really Doing?

<div class="intuition-box">
<span class="box-label">💡 The Three-Act Story</span>

The formula $\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}$ is telling a very concrete three-act story. Let us walk through it with a concrete vector and the matrix from Example 4:

$$\mathbf{A} = \begin{pmatrix} 3 & 1 \\ 0 & 2 \end{pmatrix}, \quad \mathbf{P} = \begin{pmatrix}1&-1\\0&1\end{pmatrix}, \quad \mathbf{D} = \begin{pmatrix}3&0\\0&2\end{pmatrix}, \quad \mathbf{P}^{-1} = \begin{pmatrix}1&1\\0&1\end{pmatrix}.$$

Take $\mathbf{x} = \begin{pmatrix}2\\1\end{pmatrix}$ and watch what happens step by step.
</div>

#### Step 1 — Apply $\mathbf{P}^{-1}$: Change into the eigenvector basis

$$\mathbf{P}^{-1}\mathbf{x} = \begin{pmatrix}1&1\\0&1\end{pmatrix}\begin{pmatrix}2\\1\end{pmatrix} = \begin{pmatrix}3\\1\end{pmatrix}.$$

**What is this doing?** The standard basis $\{(1,0)^T,\,(0,1)^T\}$ is not the "natural" basis for $\mathbf{A}$. The natural basis consists of the eigenvectors $\mathbf{v}_1$ and $\mathbf{v}_2$. Applying $\mathbf{P}^{-1}$ re-expresses $\mathbf{x}$ in eigenvector coordinates. The result $(3,1)^T$ means:

$$\mathbf{x} = 3\,\mathbf{v}_1 + 1\,\mathbf{v}_2 = 3\begin{pmatrix}1\\0\end{pmatrix} + 1\begin{pmatrix}-1\\1\end{pmatrix} = \begin{pmatrix}2\\1\end{pmatrix}. \checkmark$$

**Why do we need this?** In standard coordinates, $\mathbf{A}$ mixes coordinates together — the off-diagonal entry 1 causes coupling between axes. In the eigenvector basis, there is **no coupling** whatsoever. The transformation becomes completely diagonal: each axis acts independently. Think of it like rotating your coordinate system to align with the natural axes of the problem — measurements become simple.

#### Step 2 — Apply $\mathbf{D}$: Scale each eigenvector coordinate independently

$$\mathbf{D}\,(\mathbf{P}^{-1}\mathbf{x}) = \begin{pmatrix}3&0\\0&2\end{pmatrix}\begin{pmatrix}3\\1\end{pmatrix} = \begin{pmatrix}9\\2\end{pmatrix}.$$

**What is this doing?** In the eigenvector basis, $\mathbf{A}$ is trivially simple: multiply the $\mathbf{v}_1$ component by $\lambda_1 = 3$, and the $\mathbf{v}_2$ component by $\lambda_2 = 2$. Independently. No mixing. No interaction.

- The component $3$ (amount of $\mathbf{v}_1$) becomes $3 \times 3 = 9$.
- The component $1$ (amount of $\mathbf{v}_2$) becomes $2 \times 1 = 2$.

**This is the whole point.** Applying a general matrix $\mathbf{A}$ in standard coordinates is complicated. Applying a diagonal $\mathbf{D}$ in eigenvector coordinates is trivial — pure scalar multiplication on each axis independently. By changing basis first, we reduce a hard problem to an easy one.

#### Step 3 — Apply $\mathbf{P}$: Change back to the standard basis

$$\mathbf{P}\,(\mathbf{D}\,\mathbf{P}^{-1}\mathbf{x}) = \begin{pmatrix}1&-1\\0&1\end{pmatrix}\begin{pmatrix}9\\2\end{pmatrix} = \begin{pmatrix}7\\2\end{pmatrix}.$$

**What is this doing?** After the diagonal scaling, the result $(9,2)^T$ is still in eigenvector coordinates: $9\,\mathbf{v}_1 + 2\,\mathbf{v}_2$. Applying $\mathbf{P}$ converts it back to standard coordinates so we can read the answer in the usual way.

**Verification:**

$$\mathbf{A}\mathbf{x} = \begin{pmatrix}3&1\\0&2\end{pmatrix}\begin{pmatrix}2\\1\end{pmatrix} = \begin{pmatrix}7\\2\end{pmatrix}. \checkmark$$

### Summary: The Three-Step Story

<table class="step-table">
  <tr>
    <th>Step</th>
    <th>Operation</th>
    <th>What it means geometrically</th>
  </tr>
  <tr>
    <td><strong>1</strong></td>
    <td>$\mathbf{P}^{-1}$</td>
    <td>Rotate into the eigenvector coordinate system</td>
  </tr>
  <tr>
    <td><strong>2</strong></td>
    <td>$\mathbf{D}$</td>
    <td>Scale each eigenvector axis by its eigenvalue (trivially simple)</td>
  </tr>
  <tr>
    <td><strong>3</strong></td>
    <td>$\mathbf{P}$</td>
    <td>Rotate back to the standard coordinate system</td>
  </tr>
</table>

<div class="intuition-box">
<span class="box-label">💡 Two Analogies</span>

**Rubber band analogy.** Imagine a rubber band aligned diagonally. Reasoning about how it stretches in standard $x$-$y$ coordinates is awkward. But rotate your coordinate system to align with the rubber band ($\mathbf{P}^{-1}$), and the stretching is just a horizontal scale ($\mathbf{D}$). Then rotate back ($\mathbf{P}$). The physics did not change; only your viewpoint did.

**Ellipse analogy.** An ellipse in standard coordinates has a complicated equation with $xy$ cross-terms. But rotate to align with the axes of the ellipse (the eigenvectors of the associated matrix), and the equation becomes simply $\frac{x^2}{a^2}+\frac{y^2}{b^2}=1$ — perfectly diagonal. The eigenvectors *are* the axes of the ellipse; the eigenvalues determine $a$ and $b$.
</div>

### Why Diagonalisation Is Powerful: Matrix Powers

Once we have $\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}$, computing powers is trivial:

$$\mathbf{A}^k = \mathbf{P}\mathbf{D}^k\mathbf{P}^{-1}, \qquad \mathbf{D}^k = \begin{pmatrix}\lambda_1^k & & \\ & \ddots & \\ & & \lambda_n^k\end{pmatrix}.$$

**Why does this hold?**

$$\mathbf{A}^2 = (\mathbf{P}\mathbf{D}\mathbf{P}^{-1})(\mathbf{P}\mathbf{D}\mathbf{P}^{-1}) = \mathbf{P}\mathbf{D}(\mathbf{P}^{-1}\mathbf{P})\mathbf{D}\mathbf{P}^{-1} = \mathbf{P}\mathbf{D}^2\mathbf{P}^{-1},$$

and so on by induction — the $\mathbf{P}^{-1}\mathbf{P}$ terms in the middle always collapse to $\mathbf{I}$.

<div class="example-box">
<span class="box-label">Example 8 — Matrix Powers via Diagonalisation</span>

$$\mathbf{A}^{10} = \mathbf{P}\begin{pmatrix}3^{10}&0\\0&2^{10}\end{pmatrix}\mathbf{P}^{-1} = \begin{pmatrix}1&-1\\0&1\end{pmatrix} \begin{pmatrix}59049&0\\0&1024\end{pmatrix} \begin{pmatrix}1&1\\0&1\end{pmatrix}.$$

Without diagonalisation, this would require 9 full matrix multiplications. With diagonalisation, we just raise two numbers to a power.

**Geometric reading:** Applying $\mathbf{A}^{10}$ means — go into the eigenvector basis, stretch the $\mathbf{v}_1$ direction by $3^{10} = 59049$ and the $\mathbf{v}_2$ direction by $2^{10} = 1024$, then come back. The eigenvector directions are fixed; only the scale changes.
</div>

### When Is a Matrix Diagonalisable?

<div class="thm-box">
<span class="box-label">Theorem 9 — Diagonalisability Condition</span>

$\mathbf{A} \in \mathbb{R}^{n\times n}$ is diagonalisable if and only if for each eigenvalue $\lambda$:

$$\text{geometric multiplicity}(\lambda) = \text{algebraic multiplicity}(\lambda).$$

In particular, if $\mathbf{A}$ has $n$ **distinct** eigenvalues, it is automatically diagonalisable.
</div>

**Geometric reason.** We need $n$ linearly independent eigenvectors to span all of $\mathbb{R}^n$. If for some repeated eigenvalue the eigenspace is too small, we cannot build an invertible $\mathbf{P}$ — there simply are not enough independent eigenvector directions to fill all of space.

### Non-Diagonalisable Matrices: Jordan Form

<div class="def-box">
<span class="box-label">Definition 10 — Jordan Block</span>

A <strong>Jordan block</strong> for eigenvalue $\lambda$ of size $k$ is

$$J_k(\lambda) = \begin{pmatrix} \lambda & 1 & & \\ & \lambda & \ddots & \\ & & \ddots & 1 \\ & & & \lambda \end{pmatrix} \in \mathbb{R}^{k\times k}.$$

Every matrix is similar to a <strong>Jordan normal form</strong> $\mathbf{A} = \mathbf{P}\mathbf{J}\mathbf{P}^{-1}$ where $\mathbf{J}$ is block-diagonal with Jordan blocks. Diagonalisable matrices are the special case where all Jordan blocks have size 1.
</div>

### Exercises

<div class="exercise-box">
<span class="box-label">Exercise 11 — Diagonalisation</span>

**(a)** Diagonalise $\mathbf{A} = \begin{pmatrix}5&4\\1&2\end{pmatrix}$: find $\mathbf{P}$, $\mathbf{D}$, and $\mathbf{P}^{-1}$ explicitly.

**(b)** Use diagonalisation to compute $\mathbf{A}^5$. Show each of the three steps ($\mathbf{P}^{-1}$, then $\mathbf{D}^5$, then $\mathbf{P}$) separately.

**(c)** Show that if $\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}$, then $e^{\mathbf{A}} := \sum_{k=0}^\infty \frac{\mathbf{A}^k}{k!} = \mathbf{P}\,e^{\mathbf{D}}\,\mathbf{P}^{-1}$, where $e^{\mathbf{D}} = \mathrm{diag}(e^{\lambda_1},\ldots,e^{\lambda_n})$.

**(d)** Is $\mathbf{B} = \begin{pmatrix}2&1\\0&2\end{pmatrix}$ diagonalisable? Explain using geometric and algebraic multiplicity. What does the failure of diagonalisation mean geometrically?
</div>

<div class="solution-box">
<span class="box-label">Solutions</span>

**(a)** Characteristic polynomial: $(5-\lambda)(2-\lambda)-4 = \lambda^2-7\lambda+6 = (\lambda-1)(\lambda-6) = 0$.
So $\lambda_1=1$, $\lambda_2=6$. For $\lambda_1=1$: $\mathbf{v}_1=(-4,1)^T$. For $\lambda_2=6$: $\mathbf{v}_2=(1,1)^T$.

$$\mathbf{P}=\begin{pmatrix}-4&1\\1&1\end{pmatrix},\quad \mathbf{D}=\begin{pmatrix}1&0\\0&6\end{pmatrix},\quad \mathbf{P}^{-1} = \frac{1}{-5}\begin{pmatrix}1&-1\\-1&-4\end{pmatrix}.$$

**(b)** Step 1: compute $\mathbf{P}^{-1}\mathbf{x}$ to express $\mathbf{x}$ in eigenvector coordinates. Step 2: scale with $\mathbf{D}^5 = \mathrm{diag}(1^5,\, 6^5) = \mathrm{diag}(1,\, 7776)$. Step 3: apply $\mathbf{P}$ to return to standard coordinates.

$$\mathbf{A}^5 = \mathbf{P}\begin{pmatrix}1&0\\0&7776\end{pmatrix}\mathbf{P}^{-1}.$$

**(c)**

$$e^{\mathbf{A}} = \sum_{k=0}^\infty \frac{(\mathbf{P}\mathbf{D}\mathbf{P}^{-1})^k}{k!} = \sum_{k=0}^\infty \frac{\mathbf{P}\mathbf{D}^k\mathbf{P}^{-1}}{k!} = \mathbf{P}\!\left(\sum_{k=0}^\infty\frac{\mathbf{D}^k}{k!}\right)\!\mathbf{P}^{-1} = \mathbf{P}\,e^{\mathbf{D}}\,\mathbf{P}^{-1}.$$

**(d)** $\mathbf{B}$ has $\lambda=2$ with algebraic multiplicity 2. Its eigenspace $\mathrm{Null}(\mathbf{B}-2\mathbf{I}) = \mathrm{Null}\begin{pmatrix}0&1\\0&0\end{pmatrix}$ has dimension 1. Since geometric multiplicity $1 <$ algebraic multiplicity $2$, $\mathbf{B}$ is **not diagonalisable**. Geometrically: there is only one eigenvector direction (the $x$-axis), so we cannot build a full basis. The matrix has a "shearing" component that no diagonal matrix can capture.
</div>