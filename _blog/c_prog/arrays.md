---
title: "Arrays in C"
layout: single
author_profile: true
mathjax: true
---

# Mastering C Arrays: A Comprehensive Guide

> **Who is this for?** Anyone learning C who wants to deeply understand arrays — not just how to use them, but *why* they work the way they do. Each section builds intuition first, then backs it up with definitions, code, and practice.
> **Course Level:** Undergraduate CS / Systems Programming  
> **Based on:** Deitel & Deitel, *C How to Program*, Chapter 7  
---

## Table of Contents
1. [Introduction to Arrays](#1-introduction-to-arrays)
2. [Memory Layout & The Contiguity Intuition](#2-memory-layout--the-contiguity-intuition)
3. [Defining and Accessing Arrays](#3-defining-and-accessing-arrays)
4. [Initializing Arrays](#4-initializing-arrays)
5. [Best Practices: Symbolic Constants](#5-best-practices-symbolic-constants)
6. [Iterating Over Arrays with Loops](#6-iterating-over-arrays-with-loops)
7. [Using Character Arrays to Store Strings](#7-using-character-arrays-to-store-strings)
8. [Static Local Arrays vs. Automatic Local Arrays](#8-static-local-arrays-vs-automatic-local-arrays)
9. [Passing Arrays to Functions](#9-passing-arrays-to-functions)
10. [Sorting Arrays](#10-sorting-arrays)
11. [Searching Arrays](#11-searching-arrays)
12. [Multidimensional Arrays](#12-multidimensional-arrays)
13. [Variable-Length Arrays (VLAs)](#13-variable-length-arrays-vlas)
14. [Common Mistakes & Pitfalls](#14-common-mistakes--pitfalls)
15. [Exercises](#15-exercises)
16. [Quick-Reference Summary](#16-quick-reference-summary)

---

## 1. Introduction to Arrays

### What Is an Array?

**Definition:** An array is a fixed-size, ordered collection of elements that are all of the **same data type**, stored in **contiguous (side-by-side) memory locations**.

Think of an array like a row of numbered post-office boxes. Every box is the same size, they sit right next to each other, and each has a unique number (index) you use to find it. To put mail in box #3, you don't search — you go directly there.

```
Index:  [ 0 ] [ 1 ] [ 2 ] [ 3 ] [ 4 ]
Value:  [ 10] [ 20] [ 30] [ 40] [ 50]
```

### Why Are Arrays Useful?

Without arrays, storing 100 student grades would require 100 separate variables: `grade1`, `grade2`, ..., `grade100`. With an array, you just write:

```c
int grades[100];
```

And you can process all 100 values with a simple loop. Arrays are the foundation of almost every algorithm in computer science.

### Key Properties of C Arrays

| Property | Detail |
|---|---|
| **Fixed size** | Size is set at definition time and cannot change |
| **Same type** | All elements must share one data type |
| **Zero-indexed** | First element is at index `0` |
| **Contiguous** | Elements occupy adjacent memory addresses |
| **No bounds checking** | C will not stop you from going out of bounds |

---

## 2. Memory Layout & The Contiguity Intuition

### The Ruler Analogy

Imagine a ruler lying flat. Each centimetre mark is a memory address. When you declare `int c[5]`, the computer finds 5 consecutive centimetre slots, each wide enough to hold one `int` (typically 4 bytes), and reserves them all for you.

```
Address:  1000   1004   1008   1012   1016
          +------+------+------+------+------+
          | c[0] | c[1] | c[2] | c[3] | c[4] |
          +------+------+------+------+------+
```

Each slot is 4 bytes apart because `sizeof(int) == 4`. This predictable layout is what makes array access so fast — to find element `i`, the CPU simply computes:

```
address of c[i] = base address + (i x sizeof(element))
```

This is **O(1)** — constant time access, regardless of array size.

### Why Does Zero-Indexing Follow From This?

The index is actually an **offset** from the start. `c[0]` is 0 steps from the beginning, `c[1]` is 1 step, and so on. Zero-indexing is not arbitrary — it maps directly onto this address arithmetic.

---

## 3. Defining and Accessing Arrays

### Defining an Array

**Syntax:**
```c
type arrayName[size];
```

- `type` — the data type of every element (`int`, `float`, `char`, etc.)
- `arrayName` — follows standard C identifier rules (letters, digits, underscores; cannot start with a digit)
- `size` — a positive integer constant specifying the number of elements

**Examples:**

```c
int scores[10];        // 10 integers
float temperatures[7]; // 7 floats (one per day of the week)
char vowels[5];        // 5 characters
double prices[100];    // 100 doubles
```

### Accessing Array Elements

**Syntax:** `arrayName[index]`

The index must be an integer (or integer expression) in the range `0` to `size - 1`.

```c
int c[5];

c[0] = 10;   // first element
c[1] = 20;
c[2] = 30;
c[3] = 40;
c[4] = 50;   // last element — index is size-1

printf("%d\n", c[2]);  // prints 30
```

> **Crucial Rule:** The first element is always at index `0`. An array of size `N` has valid indices `0` through `N-1`. Index `N` does **not** exist.

### Using Expressions as Indices

Any integer expression works as an index:

```c
int i = 3;
c[i]       // same as c[3]
c[i - 1]   // same as c[2]
c[i * 2]   // same as c[6] — careful: may be out of bounds!
```

### Operator Precedence

The subscript operator `[]` has the **highest precedence** in C, equal to the function-call operator `()`. This means `c[i] + 1` is computed as `(c[i]) + 1`, not `c[i+1]`.

### No Bounds Checking — A Critical Warning

C will **silently** let you access memory outside the array. This is one of the most dangerous features of the language.

```c
int arr[5];
arr[10] = 999;  // UNDEFINED BEHAVIOR — writes to random memory!
arr[-1] = 0;    // Also undefined behavior!
```

This can cause crashes, corrupted data, or security vulnerabilities. **Always validate your indices.**

> **Best Practice:** When iterating, use a loop condition like `i < SIZE` (strict less-than) rather than `i <= SIZE` to avoid off-by-one errors.

---

## 4. Initializing Arrays

### Uninitialized Arrays Contain Garbage

If you declare an array without initializing it, its elements hold whatever random bytes happen to be at those memory addresses — called **garbage values**.

```c
int a[5];
printf("%d\n", a[0]);  // Could print anything: 0, -134217, 42, ...
```

Never assume an uninitialized array contains zeros.

### Initializer Lists

You can initialize an array at the point of definition:

```c
int n[5] = {32, 27, 64, 18, 95};
```

Memory layout after initialization:

```
Index: [  0  ] [  1  ] [  2  ] [  3  ] [  4  ]
Value: [  32 ] [  27 ] [  64 ] [  18 ] [  95 ]
```

### Partial Initialization

If you provide **fewer** values than the array size, the remaining elements are automatically set to `0`:

```c
int n[5] = {10, 20};
// Result: {10, 20, 0, 0, 0}
```

This is a clean trick to zero-initialize an entire array:

```c
int zeroed[100] = {0};  // All 100 elements are 0
```

### Implicit Sizing

Omit the size and the compiler counts the initializers for you:

```c
int primes[] = {2, 3, 5, 7, 11};
// Compiler sets size to 5 automatically
```

Use `sizeof` to find the number of elements at runtime:

```c
int count = sizeof(primes) / sizeof(primes[0]);  // = 5
```

> **`sizeof` idiom:** `sizeof(array) / sizeof(array[0])` is the standard C way to get the number of elements. Memorize this — you will use it constantly.

### Syntax Error: Too Many Initializers

```c
int n[3] = {1, 2, 3, 4};  // ERROR: 4 values for a 3-element array
```

### Initializing with a Loop

For large arrays, a loop is more practical:

```c
int squares[10];
for (int i = 0; i < 10; i++) {
    squares[i] = i * i;
}
// squares = {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}
```

---

## 5. Best Practices: Symbolic Constants

### The Problem with Magic Numbers

```c
int grades[30];
for (int i = 0; i < 30; i++) { ... }
```

If you hardcode `30` everywhere, you must find and change every occurrence. Miss one and you have a bug.

### The Solution: `#define`

```c
#define NUM_STUDENTS 30

int grades[NUM_STUDENTS];
for (int i = 0; i < NUM_STUDENTS; i++) { ... }
```

Now changing the class size requires editing **one line only**.

**How it works:** The C preprocessor runs before compilation and performs a text substitution — every occurrence of `NUM_STUDENTS` in your source code is replaced with `30` before the compiler ever sees it.

### Using `const` Instead

Modern C style often prefers `const` variables:

```c
const int SIZE = 10;
int arr[SIZE];
```

Unlike `#define`, `const` variables are **type-safe** and visible to the debugger.

### Never Put a Semicolon After `#define`

```c
#define SIZE 5;   // WRONG!
int arr[SIZE];    // Becomes: int arr[5;]; — syntax error!
```

The preprocessor does a dumb text replacement. It will paste the semicolon into your code everywhere `SIZE` appears.

---

## 6. Iterating Over Arrays with Loops

Loops and arrays are natural partners. The loop variable serves as the array index.

### Printing All Elements

```c
#define SIZE 5
int a[SIZE] = {10, 20, 30, 40, 50};

for (int i = 0; i < SIZE; i++) {
    printf("a[%d] = %d\n", i, a[i]);
}
```

Output:
```
a[0] = 10
a[1] = 20
a[2] = 30
a[3] = 40
a[4] = 50
```

### Computing the Sum and Average

```c
#define SIZE 5
int scores[SIZE] = {85, 92, 78, 95, 88};
int sum = 0;

for (int i = 0; i < SIZE; i++) {
    sum += scores[i];
}

double average = (double)sum / SIZE;
printf("Sum: %d, Average: %.2f\n", sum, average);
// Output: Sum: 438, Average: 87.60
```

### Finding the Maximum

```c
int max = scores[0];  // assume first is largest
for (int i = 1; i < SIZE; i++) {
    if (scores[i] > max) {
        max = scores[i];
    }
}
printf("Max: %d\n", max);  // Max: 95
```

> **Intuition:** Start by assuming the first element is the answer. Walk through the rest, updating your answer whenever you find something better. This "running champion" pattern appears everywhere in algorithms.

### Reversing an Array In-Place

```c
int left = 0, right = SIZE - 1;
while (left < right) {
    int temp   = a[left];
    a[left]    = a[right];
    a[right]   = temp;
    left++;
    right--;
}
```

> **Intuition:** Use two pointers — one starting from each end — and swap them as they march toward each other. Stop when they meet in the middle.

---

## 7. Using Character Arrays to Store Strings

### Strings in C Are Character Arrays

In C, there is no built-in `string` type. A string is simply an array of `char` values terminated by the **null character** `'\0'` (ASCII value 0).

```c
char greeting[] = "Hello";
```

Even though `"Hello"` has 5 visible characters, the array has **6 elements**:

```
Index: [ 0 ] [ 1 ] [ 2 ] [ 3 ] [ 4 ] [ 5 ]
Value: [ H ] [ e ] [ l ] [ l ] [ o ] [\0 ]
```

The null terminator `'\0'` signals the end of the string to every function that processes it (`printf`, `strlen`, `strcpy`, etc.). Without it, these functions would keep reading memory past your array — a very dangerous bug.

### Manual vs. Automatic Null Termination

The string literal `"Hello"` automatically includes `'\0'`. But if you initialize character-by-character, you must add it yourself:

```c
char name[6] = {'H', 'e', 'l', 'l', 'o', '\0'};  // correct
char bad[5]  = {'H', 'e', 'l', 'l', 'o'};         // missing '\0' — dangerous!
```

### Printing and Reading Strings

```c
char city[50] = "York";
printf("%s\n", city);   // Output: York
```

Reading from user input:

```c
char name[20];
scanf("%19s", name);   // No & needed — the array name IS already an address.
```

> **Buffer Overflow Warning:** Always limit `scanf` input with a width specifier (`%19s` for a 20-element array). Without it, a user could type 100 characters and overwrite memory beyond your array — a classic security vulnerability.

### Useful String Functions from `<string.h>`

```c
#include <string.h>

char s1[20] = "Hello";
char s2[]   = "World";

strlen(s1)        // returns 5  (does not count '\0')
strcpy(s1, s2)    // copies s2 into s1
strcat(s1, s2)    // appends s2 to end of s1
strcmp(s1, s2)    // 0 if equal, negative or positive otherwise
```

> Always make sure the destination array is large enough before calling `strcpy` or `strcat`.

---

## 8. Static Local Arrays vs. Automatic Local Arrays

### Lifetime of Local Variables

Every local variable has a **lifetime** — the period during which it exists in memory.

- **Automatic** (default): created when the function is called, destroyed when it returns.
- **Static**: created once at program startup, persists for the entire program run.

### Automatic Arrays

```c
void foo() {
    int data[5];    // created fresh each call, full of garbage
    data[0] = 42;
    // data is destroyed when foo() returns
}
```

Each call to `foo()` gets a brand-new array full of garbage values. Changes do not persist between calls.

### Static Arrays

```c
void counter() {
    static int counts[5] = {0};  // initialized ONCE, persists forever
    counts[0]++;
    printf("%d\n", counts[0]);
}

counter();  // prints 1
counter();  // prints 2
counter();  // prints 3
```

> **Intuition:** Think of a static array as a notepad that stays on your desk permanently. An automatic array is like a sticky note — you write on it during the meeting and throw it away when you leave the room.

### Summary Table

| Feature | Automatic Array | Static Array |
|---|---|---|
| Created | Each function call | Once at startup |
| Destroyed | Each function return | Never (lives until program ends) |
| Default value | Garbage | Zero |
| Use case | Temporary scratch space | Persistent counters, caches |

---

## 9. Passing Arrays to Functions

### Arrays Are Passed by Reference

When you pass an array to a function, you are **not** copying all its elements. Instead, C passes a **pointer to the first element**. This means the function can read and modify the original array.

```c
void doubleAll(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        arr[i] *= 2;    // modifies the ORIGINAL array
    }
}

int main() {
    int nums[4] = {1, 2, 3, 4};
    doubleAll(nums, 4);
    // nums is now {2, 4, 6, 8}
    return 0;
}
```

> **Intuition:** Passing an array is like giving someone the address of your house, not a copy of your house. They can go there and rearrange your furniture.

### Always Pass the Size Separately

A function receiving an array has **no way** to know how large it is. Always pass the size as a separate parameter:

```c
void printArray(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}
```

### Individual Elements Are Passed by Value

Passing a single element works differently — a copy is made:

```c
void tryToChange(int x) {
    x = 999;   // only modifies the local copy
}

int arr[3] = {1, 2, 3};
tryToChange(arr[1]);
// arr[1] is still 2
```

### Protecting Arrays with `const`

If a function should only **read** an array, not modify it, declare the parameter `const`:

```c
void printArray(const int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    // arr[0] = 99;  // COMPILE ERROR — cannot modify const array
}
```

This enforces the **principle of least privilege**: give functions only the access they need, no more.

---

## 10. Sorting Arrays

### Why Sorting Matters

Sorted data enables faster search algorithms, makes output readable, and simplifies many problems (e.g., finding duplicates, computing medians). Sorting is one of the most studied problems in computer science.

### Algorithm 1: Bubble Sort

**Intuition:** Imagine bubbles rising in water. Large values "bubble up" toward the end of the array over multiple passes. On each pass, we compare adjacent pairs and swap them if they are in the wrong order.

**Step-by-step trace** for `{5, 3, 8, 1}`:

```
Pass 1:  [5, 3, 8, 1]
  Compare 5,3 -> swap -> [3, 5, 8, 1]
  Compare 5,8 -> ok    -> [3, 5, 8, 1]
  Compare 8,1 -> swap -> [3, 5, 1, 8]   <- 8 has "bubbled" to its place

Pass 2:  [3, 5, 1, 8]
  Compare 3,5 -> ok    -> [3, 5, 1, 8]
  Compare 5,1 -> swap -> [3, 1, 5, 8]

Pass 3:  [3, 1, 5, 8]
  Compare 3,1 -> swap -> [1, 3, 5, 8]  Sorted!
```

**Full implementation:**

```c
#define SIZE 5

void bubbleSort(int a[], int size) {
    for (int pass = 0; pass < size - 1; pass++) {
        for (int i = 0; i < size - 1 - pass; i++) {
            if (a[i] > a[i + 1]) {
                int hold  = a[i];
                a[i]      = a[i + 1];
                a[i + 1]  = hold;
            }
        }
    }
}

int main() {
    int data[SIZE] = {64, 34, 25, 12, 22};
    bubbleSort(data, SIZE);

    for (int i = 0; i < SIZE; i++) {
        printf("%d ", data[i]);
    }
    // Output: 12 22 25 34 64
    return 0;
}
```

**Complexity:** O(n²) — fine for small arrays, slow for large ones.

### Algorithm 2: Selection Sort

**Intuition:** On each pass, find the smallest remaining element and swap it into its correct position. Like picking cards from a shuffled deck and placing them in order one by one.

```c
void selectionSort(int a[], int size) {
    for (int i = 0; i < size - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < size; j++) {
            if (a[j] < a[minIdx]) {
                minIdx = j;
            }
        }
        int temp   = a[i];
        a[i]       = a[minIdx];
        a[minIdx]  = temp;
    }
}
```

**Trace for `{29, 10, 14, 37, 13}`:**
```
Pass 0: min=10 at idx 1 -> swap with idx 0 -> {10, 29, 14, 37, 13}
Pass 1: min=13 at idx 4 -> swap with idx 1 -> {10, 13, 14, 37, 29}
Pass 2: min=14 at idx 2 -> no swap needed  -> {10, 13, 14, 37, 29}
Pass 3: min=29 at idx 4 -> swap with idx 3 -> {10, 13, 14, 29, 37}
```

### Algorithm 3: Insertion Sort

**Intuition:** Like sorting playing cards in your hand. You pick up one card at a time and insert it into the correct position among the cards you have already sorted.

```c
void insertionSort(int a[], int size) {
    for (int i = 1; i < size; i++) {
        int key = a[i];
        int j   = i - 1;
        while (j >= 0 && a[j] > key) {
            a[j + 1] = a[j];   // shift right
            j--;
        }
        a[j + 1] = key;        // insert in correct spot
    }
}
```

### Sorting Algorithm Comparison

| Algorithm | Best Case | Average Case | Worst Case | Stable? |
|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | No |

> For production code, use C's built-in `qsort()` from `<stdlib.h>` which implements an efficient hybrid sort.

---

## 11. Searching Arrays

### Algorithm 1: Linear Search

**Intuition:** Start at the beginning and check every element one by one until you find the target or exhaust the array. Simple but potentially slow.

```c
int linearSearch(int arr[], int size, int key) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == key) {
            return i;   // found: return the index
        }
    }
    return -1;          // not found
}
```

**Usage:**
```c
int data[] = {34, 7, 23, 32, 5, 62};
int idx = linearSearch(data, 6, 23);
// idx = 2
```

**Performance:** O(n) — must check up to n elements. Good for small or unsorted arrays.

### Algorithm 2: Binary Search

**Prerequisite:** The array must be **sorted**.

**Intuition:** Think of a dictionary. You do not read every word from page 1. Instead you open to the middle — if your word comes before the middle word, search the left half; otherwise search the right half. Each step cuts the problem in half.

**Step-by-step trace** — searching for `23` in `{2, 5, 8, 12, 16, 23, 38, 56}`:

```
low=0, high=7, mid=3  -> arr[3]=12 < 23 -> search right half
low=4, high=7, mid=5  -> arr[5]=23 == 23 -> FOUND at index 5
```

**Full implementation:**

```c
int binarySearch(int arr[], int size, int key) {
    int low  = 0;
    int high = size - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2;  // avoids integer overflow

        if (arr[mid] == key) {
            return mid;         // found
        } else if (arr[mid] < key) {
            low = mid + 1;      // search right half
        } else {
            high = mid - 1;     // search left half
        }
    }

    return -1;  // not found
}
```

> **Why `low + (high - low) / 2` instead of `(low + high) / 2`?**
> If `low` and `high` are both large integers, `low + high` can overflow. The subtraction form is always safe.

### Performance Comparison

| Array Size | Linear Search (avg comparisons) | Binary Search (max comparisons) |
|---|---|---|
| 10 | 5 | 4 |
| 1,000 | 500 | 10 |
| 1,000,000 | 500,000 | 20 |
| 1,000,000,000 | 500,000,000 | 30 |

> **The Power of Logarithms:** Binary search runs in O(log n). Each comparison eliminates half the remaining elements. That is why a billion elements only needs 30 comparisons — 2 to the power of 30 is greater than one billion.

---

## 12. Multidimensional Arrays

### The 2D Array: A Table of Values

A two-dimensional array is essentially a table with rows and columns. Think of a seating chart, a spreadsheet, or a game board.

**Declaration:**
```c
int matrix[3][4];   // 3 rows, 4 columns = 12 total elements
```

**Accessing elements:** `matrix[row][col]`

```
        col 0   col 1   col 2   col 3
row 0: [  1  ] [  2  ] [  3  ] [  4  ]
row 1: [  5  ] [  6  ] [  7  ] [  8  ]
row 2: [  9  ] [ 10  ] [ 11  ] [ 12  ]
```

### Initialization

{% raw %}
```c
int b[3][4] = {
    { 1,  2,  3,  4},   // row 0
    { 5,  6,  7,  8},   // row 1
    { 9, 10, 11, 12}    // row 2
};
```
{% endraw %}

### Iterating with Nested Loops

```c
#define ROWS 3
#define COLS 4

for (int r = 0; r < ROWS; r++) {
    for (int c = 0; c < COLS; c++) {
        printf("%4d", b[r][c]);
    }
    printf("\n");
}
```

Output:
```
   1   2   3   4
   5   6   7   8
   9  10  11  12
```

### How 2D Arrays Are Stored in Memory

Despite looking like a table, a 2D array is stored **row by row** in a single contiguous block — called **row-major order**.

```
Memory: [b[0][0]] [b[0][1]] [b[0][2]] [b[0][3]] [b[1][0]] [b[1][1]] ...
```

> **Performance tip:** Always loop over **columns in the inner loop**. This accesses memory sequentially and is CPU-cache-friendly. Swapping the loops (iterating rows in the inner loop) causes cache misses and can be significantly slower on large arrays.

### The Comma Operator Trap

```c
int a[3][3];
a[1, 2] = 5;   // Logic error! Comma operator evaluates to a[2], not a[1][2]
a[1][2] = 5;   // Correct
```

### Passing 2D Arrays to Functions

The first dimension can be omitted; all subsequent dimensions are **required** so the compiler can calculate memory offsets:

```c
void printMatrix(int arr[][4], int rows) {
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < 4; c++) {
            printf("%4d", arr[r][c]);
        }
        printf("\n");
    }
}
```

### Practical Example: Matrix Addition

{% raw %}
```c
#define N 3

void addMatrices(int a[][N], int b[][N], int result[][N]) {
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            result[i][j] = a[i][j] + b[i][j];
        }
    }
}
```
{% endraw %}

---

## 13. Variable-Length Arrays (VLAs)

### The Problem with Fixed Sizes

Sometimes you do not know the array size until the program is running:

```c
int data[100];   // What if the user has 200 items? Or just 3?
```

### VLAs to the Rescue (C99 and later)

```c
#include <stdio.h>

int main() {
    int n;
    printf("How many grades? ");
    scanf("%d", &n);

    int grades[n];    // size determined at runtime!

    for (int i = 0; i < n; i++) {
        printf("Enter grade %d: ", i + 1);
        scanf("%d", &grades[i]);
    }

    int sum = 0;
    for (int i = 0; i < n; i++) sum += grades[i];
    printf("Average: %.2f\n", (double)sum / n);

    return 0;
}
```

### Limitations of VLAs

| Feature | VLAs |
|---|---|
| Size known at compile time? | No — set at runtime |
| Can initialize with `= {0}`? | No |
| Can be `static`? | No |
| Storage location | Stack (risk of stack overflow for large n) |
| Guaranteed in C11+? | Optional — not all compilers support it |

> For large or unpredictable sizes, prefer **dynamic memory allocation** with `malloc()` and `free()` from `<stdlib.h>` — that uses the heap which has much more space.

---

## 14. Common Mistakes & Pitfalls

### Mistake 1: Off-by-One Error

```c
int arr[5];
for (int i = 0; i <= 5; i++) {   // BUG: i goes up to 5
    arr[i] = 0;                   // arr[5] is out of bounds!
}
// Fix: use i < 5
```

### Mistake 2: Forgetting the Null Terminator

```c
char s[5] = {'H', 'e', 'l', 'l', 'o'};  // No '\0'!
printf("%s\n", s);  // Reads past the array — undefined behavior
// Fix: char s[6] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

### Mistake 3: Comparing Arrays with `==`

```c
int a[] = {1, 2, 3};
int b[] = {1, 2, 3};

if (a == b) { ... }  // WRONG: compares addresses, not contents!
```

To compare arrays, compare element-by-element or use `memcmp()` from `<string.h>`.

### Mistake 4: Returning a Local Array

```c
int* badFunction() {
    int local[5] = {1, 2, 3, 4, 5};
    return local;   // DANGEROUS: local is destroyed when function returns!
}
```

The returned pointer points to memory that no longer belongs to your program.

### Mistake 5: Semicolon After `#define`

```c
#define MAX 10;     // BUG: trailing semicolon
int arr[MAX];       // Becomes: int arr[10;]; — syntax error
```

### Mistake 6: Using `sizeof` on an Array Parameter

```c
void f(int arr[]) {
    int n = sizeof(arr) / sizeof(arr[0]);  // WRONG: sizeof(arr) is sizeof(pointer), not the array
}
```

Inside a function, `arr` decays to a pointer. Always pass size as a separate parameter.

---

## 15. Exercises

### Beginner

**Exercise 1:** Declare an integer array of size 8. Use a loop to fill it with the first 8 even numbers (2, 4, 6, ..., 16). Then print them.

**Exercise 2:** Write a program that reads 5 integers from the user into an array and then prints them in reverse order.

**Exercise 3:** Given `int temps[7] = {22, 25, 19, 30, 27, 24, 21}`, write code to find and print the minimum and maximum temperatures.

**Exercise 4:** Write code that counts how many elements of an integer array are negative.

---

### Intermediate

**Exercise 5:** Write a function `int sumArray(int arr[], int size)` that returns the sum of all elements. Test it with `{3, 1, 4, 1, 5, 9, 2, 6}`.

**Exercise 6:** Write a function `int countOccurrences(int arr[], int size, int target)` that counts how many times `target` appears in the array.

**Exercise 7:** Write a program that merges two sorted arrays into a third sorted array without calling any sorting function.

**Exercise 8:** Write a function that checks whether an array is already sorted in ascending order. Return `1` if sorted, `0` if not.

**Exercise 9:** Implement insertion sort and trace it step-by-step for `{9, 5, 1, 4, 3}`.

---

### Advanced

**Exercise 10:** Implement a function that **rotates** an array left by `k` positions. For `{1,2,3,4,5}` and `k=2`, the result should be `{3,4,5,1,2}`. Solve it in O(n) time and O(1) extra space using the three-reversal trick.

**Exercise 11:** Write a program using a 2D array to represent a 3x3 tic-tac-toe board. Let two players alternate entering moves (row and column) and detect when someone wins or the board is full.

**Exercise 12:** Implement binary search recursively:

```c
int binarySearchRecursive(int arr[], int low, int high, int key);
```

**Exercise 13:** Write a function that finds the two elements in an array whose sum is closest to a given target value. For example, in `{1, 3, 4, 7, 10}` with target `15`, the answer is `5 + 10`.

---

### Conceptual Questions

1. Why does C use zero-based indexing instead of starting at 1?
2. What happens in memory when a function with a local array returns? What about a `static` local array?
3. Why must you pass the size of an array as a separate parameter to a function?
4. What is the difference between passing `arr[2]` and `arr` to a function?
5. An array has 1 million elements. You run linear search and do not find the key. How many comparisons were made? What if you had used binary search on a sorted version?
6. Why is iterating columns in the inner loop faster than iterating rows in the inner loop for a 2D array?
7. Explain why `char name[] = "Bob"` requires 4 bytes, not 3.
8. Can two arrays be swapped using `int temp = a; a = b; b = temp;`? Why or why not?

---

## 16. Quick-Reference Summary

```c
/* --- DECLARATION --- */
int arr[10];                         // 10 uninitialized ints
int arr[5] = {1, 2, 3, 4, 5};       // initialized
int arr[]  = {1, 2, 3};             // size inferred = 3
int arr[5] = {0};                    // all zeros

/* --- ACCESS --- */
arr[0]                               // first element
arr[n-1]                             // last element
sizeof(arr) / sizeof(arr[0])         // number of elements (only works in scope of declaration)

/* --- STRINGS --- */
char s[] = "hello";                  // 6 chars including '\0'
scanf("%19s", s);                    // safe input for s[20]
strlen(s)                            // length without '\0'

/* --- FUNCTIONS --- */
void f(int arr[], int size);         // pass array by reference
void f(const int arr[], int size);   // read-only array

/* --- 2D ARRAYS --- */
int m[3][4];                         // 3 rows, 4 columns
m[r][c]                              // access element at row r, col c
void f(int m[][4], int rows);        // pass 2D array to function

/* --- SYMBOLIC CONSTANTS --- */
#define SIZE 10                      // no semicolon!
const int SIZE = 10;                 // type-safe alternative

/* --- VLA (C99+) --- */
int n;
scanf("%d", &n);
int arr[n];                          // runtime-sized array

/* --- SORTING --- */
bubbleSort(arr, size);               // O(n^2), in-place
selectionSort(arr, size);            // O(n^2), in-place
insertionSort(arr, size);            // O(n^2), efficient for nearly-sorted data

/* --- SEARCHING --- */
linearSearch(arr, size, key);        // O(n), works on unsorted arrays
binarySearch(arr, size, key);        // O(log n), requires sorted array
```

---

*Arrays are the foundation of nearly every data structure you will encounter: strings, stacks, queues, hash tables, matrices, and more. Master them here and everything else becomes much easier.*
