---
title: "Eigenvalues, Eigenvectors and Diagonalisation"
layout: single
author_profile: true
mathjax: true
---

## 1. Eigenvalues and Eigenvectors

### Definitions

**Definition 1 — Eigenvalue and Eigenvector**
Let $\mathbf{A} \in \mathbb{R}^{n\times n}$. A nonzero vector $\mathbf{v} \in \mathbb{R}^n$ is called an **eigenvector** of $\mathbf{A}$ if there exists a scalar $\lambda \in \mathbb{C}$ such that $$\mathbf{A}\mathbf{v} = \lambda\mathbf{v}.$$ The scalar $\lambda$ is called the corresponding **eigenvalue**. The set of all eigenvalues of $\mathbf{A}$ is called the **spectrum** of $\mathbf{A}$, denoted $\sigma(\mathbf{A})$.
{: .notice--info}

---

### Geometric Intuition: What Do Eigenvalues Actually Mean?

**💡 Geometric Intuition**

Think of $\mathbf{A}$ as a machine that takes a vector and transforms it — rotating it, stretching it, shearing it, or flipping it.

For **most** vectors $\mathbf{x}$, after applying $\mathbf{A}$, the result $\mathbf{A}\mathbf{x}$ points in a completely different direction and has a different length. Both direction and magnitude change simultaneously.

But there exist **special** vectors — eigenvectors — for which something remarkable happens: after applying $\mathbf{A}$, the vector **still points in the same direction** (or exactly opposite). Only its length changes, by the factor $\lvert\lambda\rvert$.

The eigenvalue $\lambda$ tells you exactly what happens to those special directions:

- $\lambda > 1$: the eigenvector is **stretched** (grows longer).
- $0 < \lambda < 1$: the eigenvector is **compressed** (shrinks).
- $\lambda = 1$: the eigenvector is **left completely unchanged**.
- $\lambda < 0$: the eigenvector is **flipped** (reversed direction) and possibly scaled.
- $\lambda = 0$: the eigenvector is **collapsed to zero** — it lands in the null space of $\mathbf{A}$.

**Key insight.** Eigenvectors are the "skeleton" of a linear transformation. Once you know them, you understand the fundamental structure of how $\mathbf{A}$ acts on all of space.
{: .notice--warning}

---

### Finding Eigenvalues: The Characteristic Polynomial

From $\mathbf{A}\mathbf{v} = \lambda\mathbf{v}$, rearranging gives:

$$(\mathbf{A} - \lambda\mathbf{I})\mathbf{v} = \mathbf{0}.$$

For this to have a nontrivial solution ($\mathbf{v} \neq \mathbf{0}$), the matrix $(\mathbf{A} - \lambda\mathbf{I})$ must be **singular** (non-invertible):

$$\boxed{\det(\mathbf{A} - \lambda\mathbf{I}) = 0.}$$

This is the **characteristic equation**. The polynomial $p(\lambda) = \det(\mathbf{A}-\lambda\mathbf{I})$ of degree $n$ is the **characteristic polynomial**.

**Theorem 2 — Fundamental Theorem Applied to Eigenvalues**
Every $n \times n$ matrix has exactly $n$ eigenvalues (counting algebraic multiplicity) in $\mathbb{C}$. Over $\mathbb{R}$, complex eigenvalues always appear in conjugate pairs.
{: .notice--warning}

**Definition 3 — Eigenspace**
Given eigenvalue $\lambda$, the **eigenspace** is $$E_\lambda = \mathrm{Null}(\mathbf{A} - \lambda\mathbf{I}) = \lbrace\mathbf{v} \in \mathbb{R}^n : \mathbf{A}\mathbf{v} = \lambda\mathbf{v}\rbrace.$$ It is a subspace of $\mathbb{R}^n$. Its dimension is the **geometric multiplicity** of $\lambda$. The **algebraic multiplicity** is the multiplicity of $\lambda$ as a root of $p(\lambda)$. Always: $$\text{geometric multiplicity}(\lambda) \leq \text{algebraic multiplicity}(\lambda).$$
{: .notice--info}

**💡 Geometric Intuition — The Eigenspace as a Stable Subspace**
The eigenspace $E_\lambda$ is the collection of all eigenvectors for a given $\lambda$ (plus the zero vector). It forms a subspace — a line, a plane, or higher-dimensional analog — that $\mathbf{A}$ maps **entirely into itself**. Every vector inside $E_\lambda$ is simply scaled by $\lambda$; no vector escapes the subspace. Think of it as a "stable region" of the transformation.
{: .notice--warning}

---

### Worked Examples

**Example 4 — 2×2 Matrix**

Find the eigenvalues and eigenvectors of $\mathbf{A} = \begin{pmatrix} 3 & 1 \\ 0 & 2 \end{pmatrix}.$

**Step 1: Characteristic polynomial.**

$$p(\lambda) = \det\begin{pmatrix} 3-\lambda & 1 \\ 0 & 2-\lambda \end{pmatrix} = (3-\lambda)(2-\lambda) = \lambda^2 - 5\lambda + 6.$$

Roots: $\lambda_1 = 3$, $\lambda_2 = 2$.

**Step 2: Eigenvectors.**

For $\lambda_1 = 3$:

$$(\mathbf{A}-3\mathbf{I})\mathbf{v} = \mathbf{0} \implies \begin{pmatrix} 0 & 1 \\ 0 & -1 \end{pmatrix}\mathbf{v} = \mathbf{0} \implies v_2 = 0 \implies \mathbf{v}_1 = \begin{pmatrix}1\\0\end{pmatrix}.$$

For $\lambda_2 = 2$:

$$(\mathbf{A}-2\mathbf{I})\mathbf{v} = \mathbf{0} \implies \begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}\mathbf{v} = \mathbf{0} \implies v_1 = -v_2 \implies \mathbf{v}_2 = \begin{pmatrix}-1\\1\end{pmatrix}.$$

**Geometric meaning:** $\mathbf{v}_1 = (1,0)^T$ is stretched by factor 3 — triples in length but stays horizontal. $\mathbf{v}_2 = (-1,1)^T$ is stretched by factor 2 — doubles in length but keeps its direction. Every other vector gets both rotated and scaled; only these two directions are preserved.
{: .notice--success}

**Example 5 — Symmetric Matrix**

Find eigenvalues and eigenvectors of $\mathbf{B} = \begin{pmatrix} 4 & 2 \\ 2 & 1 \end{pmatrix}.$

**Characteristic polynomial:**

$$p(\lambda) = (4-\lambda)(1-\lambda) - 4 = \lambda^2 - 5\lambda = \lambda(\lambda-5).$$

So $\lambda_1 = 0$, $\lambda_2 = 5$.

For $\lambda_1 = 0$: $\quad \mathbf{v}_1 = \begin{pmatrix}1\\-2\end{pmatrix}.$ For $\lambda_2 = 5$: $\quad \mathbf{v}_2 = \begin{pmatrix}2\\1\end{pmatrix}.$

Note: $\mathbf{v}_1 \cdot \mathbf{v}_2 = (1)(2)+(-2)(1) = 0$. The eigenvectors are **orthogonal** — this always holds for symmetric matrices (the Spectral Theorem).
{: .notice--success}

---

### Exercises

**Exercise 6 — Computing Eigenvalues and Eigenvectors**

**(a)** Find all eigenvalues and eigenvectors of $\mathbf{A} = \begin{pmatrix}2&3\\0&5\end{pmatrix}$.

**(b)** Find all eigenvalues and eigenvectors of $\mathbf{B} = \begin{pmatrix}1&1\\1&1\end{pmatrix}$.

**(c)** Show that if $\lambda$ is an eigenvalue of $\mathbf{A}$, then $\lambda^2$ is an eigenvalue of $\mathbf{A}^2$.

**(d)** A matrix $\mathbf{P}$ satisfies $\mathbf{P}^2 = \mathbf{P}$ (a *projection*). What are the only possible eigenvalues of $\mathbf{P}$? Give a geometric reason.
{: .notice--primary}

**Solutions to Exercise 6**

**(a)** Characteristic polynomial: $(2-\lambda)(5-\lambda)=0$, so $\lambda_1=2$, $\lambda_2=5$. For $\lambda_1=2$: $\mathbf{v}_1=(1,0)^T$. For $\lambda_2=5$: $\mathbf{v}_2=(1,1)^T$.

**(b)** Characteristic polynomial: $\lambda(\lambda-2)=0$, so $\lambda_1=0$, $\lambda_2=2$. For $\lambda_1=0$: $\mathbf{v}_1=(1,-1)^T$. For $\lambda_2=2$: $\mathbf{v}_2=(1,1)^T$.

**(c)** If $\mathbf{A}\mathbf{v}=\lambda\mathbf{v}$, then $\mathbf{A}^2\mathbf{v}=\mathbf{A}(\lambda\mathbf{v})=\lambda^2\mathbf{v}$. ✓

**(d)** From $\mathbf{P}^2=\mathbf{P}$ and $\mathbf{P}\mathbf{v}=\lambda\mathbf{v}$, we get $\lambda^2=\lambda$, so $\lambda \in \lbrace 0,1\rbrace$. Geometrically: a projection either collapses a vector to zero (eigenvalue 0) or leaves it fixed in the projection subspace (eigenvalue 1).
{: .notice}

---

## 2. Diagonalisation

### Definition and Construction

**Definition 7 — Diagonalisable Matrix**
A matrix $\mathbf{A} \in \mathbb{R}^{n\times n}$ is **diagonalisable** if there exists an invertible matrix $\mathbf{P}$ and a diagonal matrix $\mathbf{D}$ such that $$\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}.$$ Equivalently, $\mathbf{A}$ is diagonalisable if and only if it has $n$ linearly independent eigenvectors.
{: .notice--info}

**How to build $\mathbf{P}$ and $\mathbf{D}$.** Place each eigenvector as a column of $\mathbf{P}$, and the corresponding eigenvalue on the diagonal of $\mathbf{D}$:

$$\mathbf{P} = \begin{pmatrix} | & | & & | \\ \mathbf{v}_1 & \mathbf{v}_2 & \cdots & \mathbf{v}_n \\ | & | & & | \end{pmatrix}, \qquad \mathbf{D} = \begin{pmatrix} \lambda_1 & & \\ & \ddots & \\ & & \lambda_n \end{pmatrix}.$$

---

### Geometric Intuition: What Is Diagonalisation Really Doing?

**💡 The Three-Act Story**
The formula $\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}$ is telling a very concrete three-act story. We use the matrix from Example 4: $\mathbf{P} = \begin{pmatrix}1&-1\\0&1\end{pmatrix}$, $\mathbf{D} = \begin{pmatrix}3&0\\0&2\end{pmatrix}$, $\mathbf{P}^{-1} = \begin{pmatrix}1&1\\0&1\end{pmatrix}$. Take $\mathbf{x} = \begin{pmatrix}2\\1\end{pmatrix}$ and watch what happens step by step.
{: .notice--warning}

#### Step 1 — Apply $\mathbf{P}^{-1}$: Change into the eigenvector basis

$$\mathbf{P}^{-1}\mathbf{x} = \begin{pmatrix}1&1\\0&1\end{pmatrix}\begin{pmatrix}2\\1\end{pmatrix} = \begin{pmatrix}3\\1\end{pmatrix}.$$

**What is this doing?** Applying $\mathbf{P}^{-1}$ re-expresses $\mathbf{x}$ in eigenvector coordinates. The result $(3,1)^T$ means $\mathbf{x} = 3\,\mathbf{v}_1 + 1\,\mathbf{v}_2$.

**Why do we need this?** In standard coordinates, $\mathbf{A}$ mixes coordinates together — the off-diagonal entry causes coupling. In the eigenvector basis, there is **no coupling** — each axis acts completely independently. Think of rotating your coordinate system to align with the natural axes of the problem.

#### Step 2 — Apply $\mathbf{D}$: Scale each coordinate independently

$$\mathbf{D}\,(\mathbf{P}^{-1}\mathbf{x}) = \begin{pmatrix}3&0\\0&2\end{pmatrix}\begin{pmatrix}3\\1\end{pmatrix} = \begin{pmatrix}9\\2\end{pmatrix}.$$

**What is this doing?** Simply multiply the $\mathbf{v}_1$ component by $\lambda_1 = 3$, and the $\mathbf{v}_2$ component by $\lambda_2 = 2$. No mixing. No interaction. The component $3$ becomes $3 \times 3 = 9$. The component $1$ becomes $2 \times 1 = 2$.

**This is the whole point.** Applying $\mathbf{A}$ in standard coordinates is hard. Applying $\mathbf{D}$ in eigenvector coordinates is trivial — pure scalar multiplication on each axis.

#### Step 3 — Apply $\mathbf{P}$: Change back to standard coordinates

$$\mathbf{P}\,(\mathbf{D}\,\mathbf{P}^{-1}\mathbf{x}) = \begin{pmatrix}1&-1\\0&1\end{pmatrix}\begin{pmatrix}9\\2\end{pmatrix} = \begin{pmatrix}7\\2\end{pmatrix}.$$

**What is this doing?** The result $(9,2)^T$ is still in eigenvector coordinates. Applying $\mathbf{P}$ converts it back to standard coordinates.

**Verification:** $\mathbf{A}\mathbf{x} = \begin{pmatrix}3&1\\0&2\end{pmatrix}\begin{pmatrix}2\\1\end{pmatrix} = \begin{pmatrix}7\\2\end{pmatrix}.$ ✓

---

### Summary: The Three Steps

| Step | Operation | What it means geometrically |
|------|-----------|----------------------------|
| 1 | $\mathbf{P}^{-1}$ | Rotate into the eigenvector coordinate system |
| 2 | $\mathbf{D}$ | Scale each axis by its eigenvalue (trivially simple) |
| 3 | $\mathbf{P}$ | Rotate back to the standard coordinate system |

**💡 Two Analogies**

**Rubber band.** Rotate your coordinate system to align with the rubber band ($\mathbf{P}^{-1}$), the stretching is just a horizontal scale ($\mathbf{D}$), then rotate back ($\mathbf{P}$). The physics did not change; only your viewpoint did.

**Ellipse.** In standard coordinates an ellipse has complicated $xy$ cross-terms. Rotate to align with its axes (the eigenvectors) and the equation becomes simply $\frac{x^2}{a^2}+\frac{y^2}{b^2}=1$. The eigenvectors *are* the axes; the eigenvalues determine $a$ and $b$.
{: .notice--warning}

---

### Why Diagonalisation Is Powerful: Matrix Powers

Once we have $\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}$, computing powers is trivial:

$$\mathbf{A}^k = \mathbf{P}\mathbf{D}^k\mathbf{P}^{-1}, \qquad \mathbf{D}^k = \begin{pmatrix}\lambda_1^k & & \\ & \ddots & \\ & & \lambda_n^k\end{pmatrix}.$$

**Why does this hold?** Because $\mathbf{P}^{-1}\mathbf{P} = \mathbf{I}$ collapses all middle terms:

$$\mathbf{A}^2 = (\mathbf{P}\mathbf{D}\mathbf{P}^{-1})(\mathbf{P}\mathbf{D}\mathbf{P}^{-1}) = \mathbf{P}\mathbf{D}(\mathbf{P}^{-1}\mathbf{P})\mathbf{D}\mathbf{P}^{-1} = \mathbf{P}\mathbf{D}^2\mathbf{P}^{-1},$$

and so on by induction.

**Example 8 — Matrix Powers via Diagonalisation**

$$\mathbf{A}^{10} = \mathbf{P}\begin{pmatrix}3^{10}&0\\0&2^{10}\end{pmatrix}\mathbf{P}^{-1} = \begin{pmatrix}1&-1\\0&1\end{pmatrix} \begin{pmatrix}59049&0\\0&1024\end{pmatrix} \begin{pmatrix}1&1\\0&1\end{pmatrix}.$$

Without diagonalisation this requires 9 full matrix multiplications. With diagonalisation we just raise two numbers to a power.

**Geometric reading:** Go into the eigenvector basis, stretch $\mathbf{v}_1$ by $3^{10} = 59049$ and $\mathbf{v}_2$ by $2^{10} = 1024$, then come back. The eigenvector directions are fixed; only the scale changes.
{: .notice--success}

---

### When Is a Matrix Diagonalisable?

**Theorem 9 — Diagonalisability Condition**
$\mathbf{A} \in \mathbb{R}^{n\times n}$ is diagonalisable if and only if for each eigenvalue $\lambda$: $$\text{geometric multiplicity}(\lambda) = \text{algebraic multiplicity}(\lambda).$$ In particular, if $\mathbf{A}$ has $n$ **distinct** eigenvalues, it is automatically diagonalisable.
{: .notice--warning}

**Geometric reason.** We need $n$ linearly independent eigenvectors to span all of $\mathbb{R}^n$. If for some repeated eigenvalue the eigenspace is too small, we cannot build an invertible $\mathbf{P}$ — there are not enough independent directions to fill all of space.

---

### Non-Diagonalisable Matrices: Jordan Form

**Definition 10 — Jordan Block**
A **Jordan block** for eigenvalue $\lambda$ of size $k$ is $$J_k(\lambda) = \begin{pmatrix} \lambda & 1 & & \\ & \lambda & \ddots & \\ & & \ddots & 1 \\ & & & \lambda \end{pmatrix} \in \mathbb{R}^{k\times k}.$$ Every matrix is similar to a **Jordan normal form** $\mathbf{A} = \mathbf{P}\mathbf{J}\mathbf{P}^{-1}$ where $\mathbf{J}$ is block-diagonal with Jordan blocks. Diagonalisable matrices are the special case where all Jordan blocks have size 1.
{: .notice--info}

---

### Exercises

**Exercise 11 — Diagonalisation**

**(a)** Diagonalise $\mathbf{A} = \begin{pmatrix}5&4\\1&2\end{pmatrix}$: find $\mathbf{P}$, $\mathbf{D}$, and $\mathbf{P}^{-1}$ explicitly.

**(b)** Use diagonalisation to compute $\mathbf{A}^5$. Show each of the three steps ($\mathbf{P}^{-1}$, then $\mathbf{D}^5$, then $\mathbf{P}$) separately.

**(c)** Show that if $\mathbf{A} = \mathbf{P}\mathbf{D}\mathbf{P}^{-1}$, then $e^{\mathbf{A}} := \sum_{k=0}^\infty \frac{\mathbf{A}^k}{k!} = \mathbf{P}\,e^{\mathbf{D}}\,\mathbf{P}^{-1}$, where $e^{\mathbf{D}} = \mathrm{diag}(e^{\lambda_1},\ldots,e^{\lambda_n})$.

**(d)** Is $\mathbf{B} = \begin{pmatrix}2&1\\0&2\end{pmatrix}$ diagonalisable? Explain using geometric and algebraic multiplicity. What does the failure of diagonalisation mean geometrically?
{: .notice--primary}

**Solutions to Exercise 11**

**(a)** Characteristic polynomial: $(\lambda-1)(\lambda-6) = 0$, so $\lambda_1=1$, $\lambda_2=6$. For $\lambda_1=1$: $\mathbf{v}_1=(-4,1)^T$. For $\lambda_2=6$: $\mathbf{v}_2=(1,1)^T$.

$$\mathbf{P}=\begin{pmatrix}-4&1\\1&1\end{pmatrix},\quad \mathbf{D}=\begin{pmatrix}1&0\\0&6\end{pmatrix},\quad \mathbf{P}^{-1} = \frac{1}{-5}\begin{pmatrix}1&-1\\-1&-4\end{pmatrix}.$$

**(b)** Step 1: compute $\mathbf{P}^{-1}\mathbf{x}$ to express $\mathbf{x}$ in eigenvector coordinates. Step 2: scale with $\mathbf{D}^5 = \mathrm{diag}(1, 7776)$. Step 3: apply $\mathbf{P}$ to return to standard coordinates.

$$\mathbf{A}^5 = \mathbf{P}\begin{pmatrix}1&0\\0&7776\end{pmatrix}\mathbf{P}^{-1}.$$

**(c)** $e^{\mathbf{A}} = \sum_{k=0}^\infty \frac{(\mathbf{P}\mathbf{D}\mathbf{P}^{-1})^k}{k!} = \mathbf{P}\!\left(\sum_{k=0}^\infty\frac{\mathbf{D}^k}{k!}\right)\!\mathbf{P}^{-1} = \mathbf{P}\,e^{\mathbf{D}}\,\mathbf{P}^{-1}.$

**(d)** $\mathbf{B}$ has $\lambda=2$ with algebraic multiplicity 2. Its eigenspace $\mathrm{Null}(\mathbf{B}-2\mathbf{I})$ has dimension 1. Since geometric multiplicity $1 <$ algebraic multiplicity $2$, $\mathbf{B}$ is **not diagonalisable**. Geometrically: there is only one eigenvector direction, so we cannot build a full basis. The matrix has a shearing component that no diagonal matrix can capture.
{: .notice}