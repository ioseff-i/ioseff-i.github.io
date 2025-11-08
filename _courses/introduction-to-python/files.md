---
title: "Introduction to Files"
layout: single
collection: courses
#parent: "Introduction to Python"
excerpt: "Reading and writing files using Python."
---


# 🐍 Understanding Loops in Python

Loops are a fundamental programming concept used to **repeat a block of code** multiple times until a certain condition is met or for a defined number of iterations.  
Python provides two main types of loops:

- [`for` loop](#-for-loop)
- [`while` loop](#-while-loop)

This guide explains both loops in detail with examples, advanced use cases, and common pitfalls.

---

## 📘 Table of Contents
1. [For Loop](#-for-loop)
   - [Basic Syntax](#basic-syntax)
   - [Looping Over Lists, Strings, and Dictionaries](#looping-over-lists-strings-and-dictionaries)
   - [Using range()](#using-range)
   - [Nested for Loops](#nested-for-loops)
   - [List Comprehension](#list-comprehension)
2. [While Loop](#-while-loop)
   - [Basic Syntax](#basic-syntax-1)
   - [While with Else](#while-with-else)
   - [Infinite Loops](#infinite-loops)
3. [Loop Control Statements](#-loop-control-statements)
   - [break](#break)
   - [continue](#continue)
   - [pass](#pass)
4. [Looping with Enumerate and Zip](#-looping-with-enumerate-and-zip)
5. [Practical Examples](#-practical-examples)
6. [Common Mistakes](#-common-mistakes)
7. [Summary Table](#-summary-table)

---

## 🔁 For Loop

The `for` loop is used to **iterate over a sequence** (like a list, tuple, dictionary, string, or range).

### Basic Syntax
```python
for variable in sequence:
    # code block
```

**Example:**
```python
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)
```

Output:
```
apple
banana
cherry
```

---

### Looping Over Lists, Strings, and Dictionaries

**List Example:**
```python
numbers = [1, 2, 3, 4]
for num in numbers:
    print(num ** 2)
```

**String Example:**
```python
for letter in "Python":
    print(letter)
```

**Dictionary Example:**
```python
student = {"name": "Alice", "age": 22, "grade": "A"}
for key, value in student.items():
    print(key, ":", value)
```

---

### Using `range()`

`range()` generates a sequence of numbers.

**Example:**
```python
for i in range(5):
    print(i)
```
Output:
```
0
1
2
3
4
```

**Custom range:**
```python
for i in range(2, 10, 2):
    print(i)
```
Output:
```
2
4
6
8
```

---

### Nested for Loops
You can place one loop inside another.

**Example:**
```python
for i in range(3):
    for j in range(2):
        print(f"i={i}, j={j}")
```

---

### List Comprehension
A concise way to create lists using loops.

**Example:**
```python
squares = [x**2 for x in range(5)]
print(squares)
```

Output:
```
[0, 1, 4, 9, 16]
```

---

## 🔄 While Loop

A `while` loop runs as long as a given condition is **True**.

### Basic Syntax
```python
while condition:
    # code block
```

**Example:**
```python
count = 0
while count < 5:
    print(count)
    count += 1
```

Output:
```
0
1
2
3
4
```

---

### While with Else

The `else` block runs **once** when the condition becomes `False`.

**Example:**
```python
x = 3
while x > 0:
    print(x)
    x -= 1
else:
    print("Loop finished!")
```

---

### Infinite Loops

If the condition **never becomes False**, the loop runs forever.

**Example:**
```python
while True:
    print("Press Ctrl+C to stop!")
```

---

## 🧭 Loop Control Statements

Python provides three keywords to control loop execution.

### `break`
Terminates the loop completely.

```python
for i in range(10):
    if i == 5:
        break
    print(i)
```

Output:
```
0
1
2
3
4
```

---

### `continue`
Skips the current iteration and moves to the next one.

```python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

Output:
```
0
1
3
4
```

---

### `pass`
A placeholder that does nothing (used for empty blocks).

```python
for i in range(3):
    pass  # Placeholder for future code
```

---

## 🔢 Looping with Enumerate and Zip

**Enumerate Example:**
```python
names = ["Alice", "Bob", "Charlie"]
for index, name in enumerate(names):
    print(index, name)
```

**Zip Example:**
```python
names = ["Alice", "Bob"]
scores = [85, 90]
for name, score in zip(names, scores):
    print(name, "scored", score)
```

---

## 🧩 Practical Examples

1. **Sum of all numbers in a list:**
```python
numbers = [1, 2, 3, 4, 5]
total = 0
for num in numbers:
    total += num
print("Sum:", total)
```

2. **Factorial using while loop:**
```python
n = 5
result = 1
while n > 0:
    result *= n
    n -= 1
print("Factorial:", result)
```

---

## ⚠️ Common Mistakes

- Forgetting to update the loop variable in a while loop → infinite loop!
- Using `=` instead of `==` in conditions.
- Modifying a list while iterating over it (use a copy instead).

---

## 📊 Summary Table

| Feature | `for` Loop | `while` Loop |
|----------|-------------|--------------|
| Iteration Type | Over sequence or range | Based on condition |
| Needs Initialization | No | Yes |
| Automatic Increment | Yes (in range) | No (manual) |
| Infinite Loop Risk | Low | High |
| Use Case | Known iterations | Unknown iterations |

---

**Author:** *Python Loops Guide*  
**License:** MIT  
**Created:** November 2025
