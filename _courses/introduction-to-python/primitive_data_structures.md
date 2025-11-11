---
title: "Primitive Data Structures in Python"
layout: single
collection: courses
parent: "Introduction to Python"
excerpt: "An overview of Python's basic built-in data structures."
---

## Mutability and Immutability

**Definition:**  
**Mutability** refers to whether the elements (or items) of an object can be changed after the object is created.  
**Immutable** objects cannot be modified once they are created.  
**Mutable** objects can be modified after creation.

- In Python, an object is the same thing as a value.  
- An **item** is one of the values in a sequence.  
- Some sequences are mutable, and some are immutable.

> **Note:** When an object is immutable, you cannot assign to any of its items using the bracket operator `[]`.  
> When an object is mutable, you can use the bracket operator on the left side of an assignment to change its contents.

---

### Mutability in Python

#### Immutable Types
- `int, float, complex, bool`
- `str, bytes`
- `tuple, frozenset`  
  → *Identity changes on "modification"*

#### Mutable Types
- `list, dict, set, bytearray`
- User-defined classes (by default)  
  → *Identity preserved on modification*

> **Memory Implications:** Immutable objects enable **interning** (memory sharing) and **hashability**.  
> Mutable objects cannot be dictionary keys or set elements.

---

### Strings Are Immutable

```python
greeting = 'Hello, world!'
greeting[0] = 'J'   # TypeError: object does not support item assignment
```

Strings are **immutable**, meaning they cannot be changed in place.  
To modify a string, create a new one:

```python
greeting = 'Hello, world!'
new_greeting = 'J' + greeting[1:]
print(new_greeting)  # Jello, world!
```

---

### Lists Are Mutable

```python
numbers = [17, 123]
numbers[1] = 5
print(numbers)   # [17, 5]
```

You can also update multiple elements using slice assignment:

```python
t = ['a', 'b', 'c', 'd', 'e', 'f']
t[1:3] = ['x', 'y']
print(t)   # ['a', 'x', 'y', 'd', 'e', 'f']
```

Key points:
- List items can be replaced or removed.  
- Modifications affect the same object (identity preserved).  
- Copy a list before modifying it for safety.

---

### Assignment and Equality

In Python:
- `a = 7` is valid, but `7 = a` is **invalid** (`SyntaxError`).
- Assignment makes two names refer to the same value but not permanently.

```python
a = 5
b = a
a = 3
```

Now `a` and `b` are no longer equal.

---

### The Meaning of Assignment

Assignment in Python stores a reference, not a mathematical equality.

```python
altitude = 320
```

- Left side → must be a variable.  
- Assignment is **not commutative**.  
- Equality in math is **permanent**, but Python assignments can change.

---

### Equality (`==`) vs Identity (`is`)

- `==` checks **value equality** (via `__eq__()`).
- `is` checks **object identity** (same memory reference).

```python
a = [1, 2, 3]
b = a
print(b is a)  # True

c = [1, 2, 3]
print(c == a)  # True
print(c is a)  # False
```

---

### Assignment and Identity

```python
a = [1, 2, 3]
b = a
print(a is b)   # True
b.append(4)
print(a)        # [1, 2, 3, 4]
```

The symbol `=` stores a reference in memory, not equality.

---

### Memory and Identity

```python
a = 256
b = 256
print(a is b)  # True

a = 1000
b = 1000
print(a is b)  # False

x = [1, 2, 3]
y = x
y.append(4)
print(x)       # [1, 2, 3, 4]
```

---

### Assignment and Copying

A **shallow copy** duplicates only the outer object, not inner elements.

```python
import copy
data = [[1, 2, 3], [4, 5, 6]]
shallow = copy.copy(data)
```

- `shallow is data → False`
- `shallow[0] is data[0] → True`

---

### Deep Copy

A **deep copy** duplicates the object *and* all objects it refers to.

```python
deep = copy.deepcopy(data)
deep is data        # False
deep[0] is data[0]  # False
```

- **Shallow copy:** outer object only  
- **Deep copy:** recursively copies all nested objects

---

### Hashing and Mutability

Hashes are integers derived from an object’s content.  
Mutable objects can’t be hashed reliably because their contents can change.

- Immutable → hashable  
- Mutable → unhashable

---

### Identity vs Hash Value

```python
a = 1000
b = int("1000")

print(a == b)           # True
print(a is b)           # False
print(hash(a) == hash(b))  # True
```

`id()` → memory identity  
`hash()` → value-based integer (only stable for immutable objects)

---

### Relations Between Mutability, Identity, and Hashing

- Immutable objects (e.g., int, str, tuple) → hashable  
- Mutable objects (e.g., list, dict) → not hashable  
- `is` → same memory object  
- `==` → equal value  
- `id()` → memory identity  
- `hash()` → content hash (immutable only)

> **Summary:**  
> Mutability affects equality, identity, and hashability.  
> Mutable objects can change identity or content; immutable ones cannot.






## Data Structures in Python

Python provides several built-in data structures to store and organize data efficiently.

- Data structures differ in two key aspects:
  - Whether they are **mutable** or **immutable**.
  - Whether they are **ordered** or **unordered**.
- Mutable structures can be changed after creation (e.g., lists, sets, dictionaries).
- Immutable structures cannot be modified after creation (e.g., tuples, strings, frozensets).

> **Definition:**  
> **Sequence:** An ordered set; a set of values where each value is identified by an integer index.

---

### Ordered and Unordered Data Structures

In Python, data structures may or may not maintain the order of their elements.

#### Ordered Data Structures
An **ordered data structure** preserves the order of insertion.  
Elements retain their position and can be accessed by index.

**Examples:**
- `list` – mutable, ordered, allows indexing and slicing.  
- `tuple` – immutable, ordered collection.  
- `string` – immutable, ordered sequence of characters.  
- `OrderedDict` (from `collections`) – dictionary that maintains insertion order.

---

### Unordered Data Structures

An **unordered data structure** does not preserve element order.  
Elements are stored in an unpredictable sequence.

**Common Uses:**
- Uniqueness of elements matters more than order.
- Fast membership testing or lookups are required.

**Examples:**
- `set` – unordered collection of unique elements.  
- `frozenset` – immutable version of a set.  
- `dict` – keys are unordered (though insertion order is preserved since Python 3.7).

---

### Lists as Mappings

You can think of a list as a relationship between **indices** and **elements**.  
This relationship is called a **mapping**; each index “maps to” one of the elements.

- Lists are represented by boxes with elements inside.
- Each variable refers to a list object that stores elements in sequence.

> Example:  
> `cheeses` refers to a list with three elements indexed 0, 1, and 2.  
> `numbers` contains two elements, one of which has been reassigned.  
> `empty` refers to an empty list.

---

### Concatenation and Repetition

The `+` operator concatenates lists.

```python
a = [1, 2, 3]
b = [4, 5, 6]
c = a + b
print(c)  # [1, 2, 3, 4, 5, 6]
```

The `*` operator repeats a list a given number of times.

```python
[0] * 4          # [0, 0, 0, 0]
[1, 2, 3] * 3    # [1, 2, 3, 1, 2, 3, 1, 2, 3]
```

---

### List Comprehensions

Python provides **list comprehensions**, a compact syntax combining mapping and filtering.

```python
# Map: squares
squares = [i**2 for i in range(10)]

# Filter: even squares
even_squares = [i**2 for i in range(10) if i % 2 == 0]

# Nested example
matrix = [[i*j for j in range(3)] for i in range(3)]
```

---

### Deleting Elements

#### Using `pop()`
```python
t = ['a', 'b', 'c']
x = t.pop(1)
print(t)  # ['a', 'c']
print(x)  # b
```

#### Using `del`
```python
t = ['a', 'b', 'c']
del t[1]
print(t)  # ['a', 'c']
```

#### Using `remove()`
```python
t = ['a', 'b', 'c']
t.remove('b')
print(t)  # ['a', 'c']
```

`remove()` deletes by value (not index) and returns `None`.

---

### Deleting Multiple Elements

To remove more than one element, use slicing with `del`:

```python
t = ['a', 'b', 'c', 'd', 'e', 'f']
del t[1:5]
print(t)  # ['a', 'f']
```

---

### List Arguments and References

When you pass a list to a function, the function gets a **reference** to the list.

```python
def delete_head(t):
    del t[0]

letters = ['a', 'b', 'c']
delete_head(letters)
print(letters)  # ['b', 'c']
```

The function modifies the same list object in memory.

---

### Lists: Ordered & Mutable

```python
# Creating lists
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]

# Common operations
numbers.append(6)      # Add to end
numbers.insert(0, 0)   # Insert at position
numbers.remove(3)      # Remove first occurrence
popped = numbers.pop() # Remove and return last

# Slicing
subset = numbers[1:4]  # Elements 1 to 3
reversed_list = numbers[::-1]
```

---

### List Comprehensions (Summary)

```python
# Traditional approach
squares = []
for i in range(10):
    squares.append(i ** 2)

# List comprehension (Pythonic!)
squares = [i**2 for i in range(10)]

# With condition
even_squares = [i**2 for i in range(10) if i % 2 == 0]

# Nested comprehension
matrix = [[i*j for j in range(3)] for i in range(3)]
```

---

### Sets: Unordered & Unique

```python
# Creating sets
fruits = {"apple", "banana", "cherry"}
numbers = set([1, 2, 2, 3, 3, 4])  # {1, 2, 3, 4}

# Operations
fruits.add("orange")
fruits.remove("banana")  # Error if not exists
fruits.discard("grape")  # No error if not exists

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
print(a | b)  # Union: {1, 2, 3, 4, 5, 6}
print(a & b)  # Intersection: {3, 4}
print(a - b)  # Difference: {1, 2}
```





### Tuples: Immutable Sequences

A **tuple** is a sequence of values, similar to a list, but **immutable**.  
Tuples are indexed by integers and can contain any data type.

```python
t = 'a', 'b', 'c', 'd', 'e'
t = ('a', 'b', 'c', 'd', 'e')
```

Parentheses are optional but improve readability.  
To create a single-element tuple, include a trailing comma:

```python
t1 = ('a',)
type(t1)  # <class 'tuple'>

t2 = ('a')
type(t2)  # <class 'str'>
```

---

### Tuple Operations and Immutability

Most list operations also work on tuples:

```python
t = ('a', 'b', 'c', 'd', 'e')
print(t[0])    # 'a'
print(t[1:3])  # ('b', 'c')
```

However, tuples are immutable:

```python
t[0] = 'A'
# TypeError: 'tuple' object does not support item assignment
```

To modify, you must create a new tuple:

```python
t = ('A',) + t[1:]
print(t)  # ('A', 'b', 'c', 'd', 'e')
```

---

### Tuple Assignment and Multiple Values

Tuples allow elegant **multiple assignments**:

```python
a, b = 1, 2
a, b = b, a
print(a, b)  # 2 1
```

Tuple unpacking works with any sequence type:

```python
addr = 'monty@python.org'
uname, domain = addr.split('@')
print(uname)   # monty
print(domain)  # python.org
```

This mechanism is called **tuple assignment**.

---

### Dictionaries: Key-Value Data Structures

A **dictionary** is like a list but more general:  
indices (called **keys**) can be of almost any immutable type.

You can think of a dictionary as a **mapping** between a set of keys and values.  
Each key maps to one value, called a **key-value pair**.

```python
eng2sp = dict()
print(eng2sp)  # {}
```

Curly braces `{}` represent an empty dictionary.

---

### Dictionary Anatomy and Creation

#### Adding Items

```python
eng2sp['one'] = 'uno'
print(eng2sp)  # {'one': 'uno'}
```

#### Anatomy
- **Keys:** unique identifiers  
- **Values:** associated data  
- **Colon (:):** separates key from value  
- **Comma (,):** separates pairs  
- **Braces {}:** enclose all pairs

#### Multiple Items

```python
eng2sp = {'one': 'uno', 'two': 'dos', 'three': 'tres'}
print(eng2sp)
# {'one': 'uno', 'three': 'tres', 'two': 'dos'}
```

The order of pairs is not guaranteed.

---

### Accessing and Using Dictionaries

Access by key, not index:

```python
print(eng2sp['two'])  # 'dos'
```

If a key doesn’t exist:

```python
print(eng2sp['four'])
# KeyError: 'four'
```

#### Other Operations

```python
len(eng2sp)     # 3
'one' in eng2sp # True
'uno' in eng2sp # False

vals = eng2sp.values()
'uno' in vals   # True
```

---

### Looping Through Dictionaries

When iterating, dictionaries traverse **keys** by default:

```python
def print_hist(h):
    for c in h:
        print(c, h[c])

h = {'a': 1, 'p': 1, 'r': 2, 't': 1, 'o': 1}
print_hist(h)
# a 1
# p 1
# r 2
# t 1
# o 1
```

> **Exercise:** Use `keys()` to print dictionary items in alphabetical order.

---

### Reverse Lookup

A **lookup** finds a value for a given key.  
A **reverse lookup** finds a key for a given value.

```python
def reverse_lookup(d, v):
    for k in d:
        if d[k] == v:
            return k
    raise ValueError
```

Example:

```python
h = {'a': 1, 'p': 1, 'r': 2, 't': 1, 'o': 1}
print(reverse_lookup(h, 2))  # r
print(reverse_lookup(h, 3))  # ValueError
```

---

### Inverting Dictionaries and Hashability

Lists can be **values** but not **keys** because keys must be **hashable**.

```python
def invert_dict(d):
    inv = dict()
    for key in d:
        val = d[key]
        if val not in inv:
            inv[val] = [key]
        else:
            inv[val].append(key)
    return inv
```

Example:

```python
hist = {'a': 1, 'p': 1, 'r': 2, 't': 1, 'o': 1}
inv = invert_dict(hist)
print(inv)  # {1: ['a', 'p', 't', 'o'], 2: ['r']}
```

---

### Inverting Dictionaries and Hashability (cont’d)

Using a list as a key causes an error:

```python
d = dict()
t = [1, 2, 3]
d[t] = 'oops'
# TypeError: list objects are unhashable
```

#### Key Points
- Dictionaries store **key-value pairs**
- Known as *associative arrays* or *hash maps*
- Store data in an **unordered** collection
- Keys must be **unique** and **immutable**
- Values can be of any data type

**Real-world analogies:**
- Phonebook: *name → number*
- Dictionary: *word → definition*

---

### Dictionary Methods Summary

| Method | Description |
|--------|--------------|
| `get(key, default)` | Retrieve value or return default |
| `del dict[key]` | Delete a key-value pair |
| `update(dict)` | Add/update multiple pairs |
| `keys()` | Get all keys |
| `values()` | Get all values |
| `items()` | Get all (key, value) tuples |

> **Remember:** Dictionaries are mutable, unordered, and incredibly powerful for data organization.

---

### Practical Example: Word Counter

```python
text = "apple banana apple cherry banana apple"
words = text.split()
word_count = {}

for word in words:
    if word in word_count:
        word_count[word] += 1
    else:
        word_count[word] = 1

print(word_count)
# {'apple': 3, 'banana': 2, 'cherry': 1}

# Cleaner approach using get()
for word in words:
    word_count[word] = word_count.get(word, 0) + 1
```







### Groupwork: Student Exam Database

**Group:** Work in your assigned team (e.g., **Group E**).  
Each group will create and analyze a small dataset using Python’s **lists**, **tuples**, **sets**, and **dictionaries**.

#### 🧩 Task: Build a Student Exam Database

1. Each group member provides their anticipated exam results in four subjects:  
   `Math`, `Computer Science`, `Physics`, and `Chemistry`.

2. Store the data as a list of **tuples**:  
   each tuple represents one student and contains their name and a dictionary of subject–score pairs.

```python
students = [
    ("Alice", {"Math": 85, "CS": 90, "Physics": 78, "Chemistry": 88}),
    ("Bob", {"Math": 92, "CS": 80, "Physics": 75, "Chemistry": 84}),
]
```

---

### Groupwork: Data Analysis Tasks

Using your group’s dataset:

1. Compute the **average score** per subject across all students.  
2. Create a **set** of all subjects in which any student scored above 90.  
3. Find the **highest scorer** in each subject.  
4. Create a **dictionary** mapping each student to their average score.  
5. Sort students (using a `list`) by their average score in descending order.

#### Extra Challenge
Convert your final results into tuples and print a formatted summary, for example:

```python
('Alice', 85.25), ('Bob', 82.75), ('Carol', 90.50)
```

**Estimated Time:** 60–75 minutes.

---

### Further Reading and References

- **Primary Text Reference:**  
  *Python for Software Design: How to Think Like a Computer Scientist*  
  by **Allen B. Downey**, Olin College of Engineering.

- **Supplementary Resource:**  
  [GeeksforGeeks](https://www.geeksforgeeks.org/) — practical explanations and examples on Python data structures and operations.

These materials provide deeper discussion on:
- Dictionaries as mappings and their internal representation  
- Hashable types and immutability  
- List, tuple, and set operations  
- Data organization and efficiency in Python
