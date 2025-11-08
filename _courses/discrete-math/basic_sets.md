---
title: "Introduction to Set Theory"
layout: single
collection: courses
parent: "Basics of Discrete Math"
excerpt: "Introduction to Set Theory"
---



# 🧮 Introduction to Set Theory

Set theory is a branch of mathematics that studies **sets**, which are collections of objects. These objects can be anything: numbers, symbols, or even other sets.  
Set theory forms the **foundation of modern mathematics**, influencing logic, probability, statistics, and computer science.

---

## 📘 Table of Contents

1. [Definition of a Set](#definition-of-a-set)
2. [Set Notation](#set-notation)
3. [Types of Sets](#types-of-sets)
4. [Set Relationships](#set-relationships)
5. [Set Operations](#set-operations)
6. [Venn Diagrams](#venn-diagrams)
7. [Laws of Set Theory](#laws-of-set-theory)
8. [Practical Applications](#practical-applications)
9. [Summary Table](#summary-table)

---

## 📗 Definition of a Set

A **set** is a *well-defined collection* of distinct elements or members.  
Sets are usually represented by **curly braces `{}`**.

**Example:**
```text
A = {1, 2, 3, 4, 5}
B = {"apple", "banana", "cherry"}
```

Here:
- `A` is a set of numbers.
- `B` is a set of strings.

---

## ✍️ Set Notation

| Symbol | Meaning | Example |
|---------|----------|----------|
| `∈` | Element of | 3 ∈ A means 3 is in A |
| `∉` | Not an element of | 6 ∉ A |
| `⊂` | Proper subset | {1,2} ⊂ {1,2,3} |
| `⊆` | Subset | {1,2} ⊆ {1,2,3} |
| `⊃` | Proper superset | {1,2,3} ⊃ {1,2} |
| `⊇` | Superset | {1,2,3} ⊇ {1,2} |
| `∅` | Empty set | ∅ = {} |
| `U` | Universal set | Set of all possible elements |

---

## 🧩 Types of Sets

1. **Empty Set (∅):** Contains no elements.  
   Example: `A = {}`

2. **Finite Set:** Has a countable number of elements.  
   Example: `B = {1, 2, 3}`

3. **Infinite Set:** Has an uncountable number of elements.  
   Example: `C = {x | x ∈ ℕ}` (all natural numbers)

4. **Subset:** Every element of one set is also in another.  
   Example: `{1, 2} ⊆ {1, 2, 3}`

5. **Universal Set (U):** The set containing all elements under consideration.  
   Example: `U = {1,2,3,4,5,6}`

6. **Power Set:** The set of all subsets of a given set.  
   Example: If `A = {1,2}`, then `P(A) = {∅, {1}, {2}, {1,2}}`

7. **Disjoint Sets:** Two sets with no common elements.  
   Example: `{1,2,3}` and `{4,5,6}`

---

## ⚖️ Set Relationships

**Equality of Sets:**  
Two sets are equal if they have exactly the same elements.
```text
A = {1, 2, 3}
B = {3, 1, 2}
A = B ✅
```

**Subset Relationship:**  
`A ⊆ B` means every element of A is also in B.

**Proper Subset:**  
`A ⊂ B` means A is a subset of B, but not equal to B.

---

## ➕ Set Operations

### 1. Union ( ∪ )
Combines all elements from both sets.
```text
A ∪ B = {x | x ∈ A or x ∈ B}
```
Example:
```text
A = {1,2,3}, B = {3,4,5}
A ∪ B = {1,2,3,4,5}
```

### 2. Intersection ( ∩ )
Common elements in both sets.
```text
A ∩ B = {x | x ∈ A and x ∈ B}
```
Example:
```text
A ∩ B = {3}
```

### 3. Difference ( − )
Elements in A but not in B.
```text
A − B = {x | x ∈ A and x ∉ B}
```
Example:
```text
A − B = {1,2}
```

### 4. Complement ( A′ )
Elements not in A, with respect to the universal set.
```text
A′ = {x | x ∈ U and x ∉ A}
```

---

## 🔵 Venn Diagrams

Venn diagrams visually represent relationships between sets.

Example (A and B):
```
        ________
       /   A ∩ B  \
      / A       B  \
     /______________\
```
- Overlapping area = A ∩ B  
- Combined area = A ∪ B  
- Outside both = (A ∪ B)'

---

## ⚙️ Laws of Set Theory

| Law | Formula | Description |
|------|----------|-------------|
| **Commutative Law** | A ∪ B = B ∪ A, A ∩ B = B ∩ A | Order doesn’t matter |
| **Associative Law** | (A ∪ B) ∪ C = A ∪ (B ∪ C) | Grouping doesn’t matter |
| **Distributive Law** | A ∩ (B ∪ C) = (A ∩ B) ∪ (A ∩ C) | Distribution over union/intersection |
| **Identity Law** | A ∪ ∅ = A, A ∩ U = A | Neutral elements |
| **Complement Law** | A ∪ A′ = U, A ∩ A′ = ∅ | Complements |
| **Idempotent Law** | A ∪ A = A, A ∩ A = A | Repetition has no effect |
| **Domination Law** | A ∪ U = U, A ∩ ∅ = ∅ | Extreme cases |

---

## 💡 Practical Applications

- **Database systems:** Querying and combining datasets.
- **Probability:** Events are treated as sets.
- **Programming:** Python’s `set()` data structure is based on set theory.
- **Logic and AI:** Knowledge representation and reasoning use set relationships.

**Example in Python:**
```python
A = {1, 2, 3}
B = {3, 4, 5}
print(A | B)  # Union
print(A & B)  # Intersection
print(A - B)  # Difference
```

---

## 📊 Summary Table

| Concept | Symbol | Example | Result |
|----------|---------|----------|--------|
| Union | ∪ | {1,2} ∪ {2,3} | {1,2,3} |
| Intersection | ∩ | {1,2} ∩ {2,3} | {2} |
| Difference | − | {1,2,3} − {2} | {1,3} |
| Complement | ′ | A′ | Elements not in A |
| Subset | ⊆ | {1,2} ⊆ {1,2,3} | True |

---

**Author:** *Set Theory Reference Guide*  
**License:** MIT  
**Created:** November 2025
