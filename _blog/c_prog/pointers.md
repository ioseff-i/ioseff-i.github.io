---
title: "Pointers in C"
layout: single
author_profile: true
mathjax: true
---

# C Pointers: Zero to Mastery

> **Course Level:** Undergraduate CS / Systems Programming  
> **Based on:** Deitel & Deitel, *C How to Program*, Chapter 7  
> **Prerequisites:** Basic C syntax, variables, functions, arrays

---

## Table of Contents

1. [Introduction — Why Pointers Matter](#1-introduction)
2. [Pointer Variables: Definition & Initialization](#2-pointer-variables-definition--initialization)
3. [Pointer Operators: `&` and `*`](#3-pointer-operators--and-)
4. [Pass-by-Reference with Pointers](#4-pass-by-reference-with-pointers)
5. [The `const` Qualifier with Pointers](#5-the-const-qualifier-with-pointers)
6. [Bubble Sort Using Pass-by-Reference](#6-bubble-sort-using-pass-by-reference)
7. [The `sizeof` Operator](#7-the-sizeof-operator)
8. [Pointer Arithmetic](#8-pointer-arithmetic)
9. [Pointers and Arrays](#9-pointers-and-arrays)
10. [Arrays of Pointers (String Arrays)](#10-arrays-of-pointers-string-arrays)
11. [Case Study: Card Shuffling and Dealing](#11-case-study-card-shuffling-and-dealing)
12. [Pointers to Functions](#12-pointers-to-functions)
13. [Secure C Programming with Pointers](#13-secure-c-programming-with-pointers)
14. [Common Mistakes Quick Reference](#14-common-mistakes-quick-reference)
15. [Practice Exercises](#15-practice-exercises)

---

## 1. Introduction

Pointers are among **the most powerful — and most difficult — features of C**. Misusing them causes bugs that are hard to trace. Mastering them unlocks:

- **Pass-by-reference:** letting functions modify variables in the caller.
- **Dynamic data structures:** linked lists, stacks, queues, trees — structures that grow and shrink at runtime.
- **Efficient function design:** passing large data without copying it.
- **Function pointers:** passing behavior as a parameter (callbacks, menu systems).

> ⚠️ **Safety note:** Pointer misuse — whether accidental or intentional — is a leading cause of security vulnerabilities (buffer overflows, NULL dereferences, dangling pointers). Always initialize your pointers.

---

## 2. Pointer Variables: Definition & Initialization

### What is a pointer?

A **normal variable** stores a value directly.  
A **pointer variable** stores the *memory address* of another variable that holds a value.

```
count = 7          →  [ 7 ]         ← direct reference
countPtr → count   →  [ • ] → [ 7 ] ← indirect reference (indirection)
```

Referencing a value through a pointer is called **indirection**.

---

### Declaring a Pointer

```c
int *countPtr;   // countPtr is a pointer to an int
int count;       // count is a plain int (NOT a pointer)
```

Read pointer declarations **right to left**:  
`int *countPtr` → *"countPtr is a pointer to int"*

> ⚠️ **Common Error:** The `*` does **not** distribute across a declaration.
>
> ```c
> int *xPtr, yPtr;   // WRONG: yPtr is an int, not a pointer!
> int *xPtr, *yPtr;  // CORRECT: both are int pointers
> ```
>
> **Best practice:** Declare one variable per line to avoid this trap.

---

### Initializing a Pointer

A pointer may be initialized to:

| Value | Meaning |
|---|---|
| `NULL` | Points to nothing (preferred) |
| `0` | Equivalent to `NULL` |
| An address | Points to a specific variable |

```c
int *p = NULL;    // safe: points to nothing
int *q = 0;       // equivalent, but NULL is clearer
int x = 5;
int *r = &x;      // r points to x
```

> **Rule:** `0` is the **only** integer that can be assigned directly to a pointer.

> 🛡️ **Error-Prevention Tip:** Always initialize pointers. An uninitialized pointer contains a garbage address — dereferencing it causes undefined behavior or crashes.

---

## 3. Pointer Operators: `&` and `*`

Two unary operators are central to pointer usage:

| Operator | Name | What it does |
|---|---|---|
| `&` | Address-of | Returns the **address** of its operand |
| `*` | Indirection (dereference) | Returns the **value** at the address stored in the pointer |

### The Address-of Operator `&`

```c
int y = 5;
int *yPtr;
yPtr = &y;   // yPtr now holds the address of y
```

In memory (example addresses):

```
Location 500000: yPtr → 600000
Location 600000: y    = 5
```

> ⚠️ The operand of `&` must be a **variable** — you cannot take the address of a constant or an expression like `&(x + 1)`.

---

### The Dereference Operator `*`

```c
printf("%d", *yPtr);   // prints 5 — the value y holds
```

Using `*` in this way is called **dereferencing a pointer**.

---

### Demonstrating `&` and `*` Together

```c
#include <stdio.h>

int main(void) {
    int a = 7;
    int *aPtr = &a;

    printf("Address of a:     %p\n", &a);
    printf("Value of aPtr:    %p\n", aPtr);   // same address
    printf("Value of a:       %d\n", a);
    printf("Value of *aPtr:   %d\n", *aPtr);  // same value

    // & and * are inverses of each other:
    printf("&*aPtr = %p\n", &*aPtr);   // same as &a
    printf("*&aPtr = %p\n", *&aPtr);   // same as aPtr
}
```

**Key insight:** `&` and `*` are **complements** — applying both (in either order) returns you to where you started.

> ⚠️ **Common Error:** Dereferencing a pointer that has not been initialized, or that points to an invalid location, causes either a crash or silent data corruption.

---

## 4. Pass-by-Reference with Pointers

### The Problem with Pass-by-Value

In C, **all arguments are passed by value** — the function gets a copy. Modifying the copy does not affect the original:

```c
int cubeByValue(int n) {
    return n * n * n;   // modifies local copy only
}

int main(void) {
    int number = 5;
    number = cubeByValue(number);   // must reassign return value
    printf("%d\n", number);         // 125
}
```

---

### Achieving Pass-by-Reference

To let a function **modify the caller's variable**, pass its **address** using `&`, and receive it with a pointer parameter, then dereference with `*`:

```c
void cubeByReference(int *nPtr) {
    *nPtr = *nPtr * *nPtr * *nPtr;   // modifies the caller's variable
}

int main(void) {
    int number = 5;
    cubeByReference(&number);    // pass address
    printf("%d\n", number);      // 125
}
```

**Step-by-step trace:**

```
Before call:    number = 5,    nPtr = undefined
After call arrives: nPtr → number (= 5)
After cubing:   *nPtr = 125   →  number in main = 125
```

---

### One-Dimensional Arrays Are Already Passed by Reference

When you pass an array to a function, C automatically passes the address of its first element — no `&` needed:

```c
void process(int arr[], int size);   // arr is really int *arr
void process(int *arr, int size);    // equivalent form
```

The compiler treats these two parameter forms **identically**.

> 🛡️ **Best practice:** Use pass-by-value unless the function explicitly needs to modify the caller's variable. This is the **principle of least privilege**.

---

## 5. The `const` Qualifier with Pointers

`const` tells the compiler: *"this value must not be modified."* There are **four combinations** when using `const` with pointers:

---

### 5.1 Non-constant Pointer to Non-constant Data

**Declaration:** `char *sPtr`  
**Can modify:** both the pointer and the data  
**Use case:** A function that processes and modifies a string.

```c
void convertToUppercase(char *sPtr) {
    while (*sPtr != '\0') {
        *sPtr = toupper(*sPtr);   // modify the character
        ++sPtr;                   // advance the pointer
    }
}
```

---

### 5.2 Non-constant Pointer to Constant Data

**Declaration:** `const char *sPtr`  
Read as: *"sPtr is a pointer to a constant char"*  
**Can modify:** the pointer (where it points), but **not** the data it points to  
**Use case:** A read-only traversal of a string.

```c
void printCharacters(const char *sPtr) {
    for (; *sPtr != '\0'; ++sPtr) {
        printf("%c", *sPtr);   // read only — no modification
    }
}
```

Attempting `*sPtr = 'X';` inside this function causes a **compile error**.

---

### 5.3 Constant Pointer to Non-constant Data

**Declaration:** `int * const ptr = &x`  
Read as: *"ptr is a constant pointer to int"*  
**Can modify:** the data, but **not** where the pointer points  
**Use case:** Default behavior of an array name — always anchored to the first element.

```c
int x = 10, y = 20;
int * const ptr = &x;
*ptr = 7;    // OK — data can change
ptr = &y;    // ERROR — ptr is const; cannot reassign
```

---

### 5.4 Constant Pointer to Constant Data

**Declaration:** `const int * const ptr = &x`  
Read as: *"ptr is a constant pointer to a constant int"*  
**Can modify:** neither the pointer nor the data  
**Use case:** Maximum protection — passing arrays to functions that only read.

```c
const int * const ptr = &x;
*ptr = 7;    // ERROR — data is const
ptr = &y;    // ERROR — pointer is const
```

---

### Summary Table

| Declaration | Pointer modifiable? | Data modifiable? |
|---|:---:|:---:|
| `int *p` | ✅ | ✅ |
| `const int *p` | ✅ | ❌ |
| `int * const p` | ❌ | ✅ |
| `const int * const p` | ❌ | ❌ |

> 🛡️ **Performance tip:** Passing large structs as `const T *` gives you the speed of pass-by-reference while preventing accidental modification — best of both worlds.

---

## 6. Bubble Sort Using Pass-by-Reference

This example reinforces pass-by-reference by splitting bubble sort into two functions: `bubbleSort` and `swap`.

```c
#include <stdio.h>
#define SIZE 10

void bubbleSort(int * const array, const size_t size);

int main(void) {
    int a[SIZE] = {2, 6, 4, 8, 10, 12, 89, 68, 45, 37};

    puts("Original order:");
    for (size_t i = 0; i < SIZE; ++i) printf("%4d", a[i]);

    bubbleSort(a, SIZE);

    puts("\nAscending order:");
    for (size_t i = 0; i < SIZE; ++i) printf("%4d", a[i]);
    puts("");
}

void bubbleSort(int * const array, const size_t size) {
    void swap(int *element1Ptr, int *element2Ptr);  // local prototype

    for (unsigned int pass = 0; pass < size - 1; ++pass)
        for (size_t j = 0; j < size - 1; ++j)
            if (array[j] > array[j + 1])
                swap(&array[j], &array[j + 1]);  // pass addresses
}

void swap(int *element1Ptr, int *element2Ptr) {
    int hold = *element1Ptr;
    *element1Ptr = *element2Ptr;
    *element2Ptr = hold;
}
```

**Key points:**
- Individual array elements are scalars — they are passed **by value** unless you explicitly use `&`.
- `swap` receives addresses and uses `*` to exchange the actual values in memory.
- `size` is declared `const` because `bubbleSort` never needs to alter it — principle of least privilege.
- The local prototype for `swap` inside `bubbleSort` restricts its proper use to that function only.

---

## 7. The `sizeof` Operator

`sizeof` is a **compile-time** unary operator that returns the number of bytes occupied by a variable or type.

```c
float array[20];
printf("%u\n", sizeof(array));       // 80  (20 elements × 4 bytes each)
printf("%u\n", sizeof(array[0]));    // 4   (one float)

// Number of elements:
size_t n = sizeof(array) / sizeof(array[0]);   // 20
```

### Typical sizes (platform-dependent)

| Type | Typical size |
|---|---|
| `char` | 1 byte |
| `short` | 2 bytes |
| `int` | 4 bytes |
| `long` | 4 or 8 bytes |
| `long long` | 8 bytes |
| `float` | 4 bytes |
| `double` | 8 bytes |
| pointer | 4 or 8 bytes |

> ⚠️ **Critical trap:** When a pointer to an array is passed to a function, `sizeof` returns the **size of the pointer** (4 or 8 bytes), **not** the size of the array. Always pass the array size separately.

```c
size_t getSize(float *ptr) {
    return sizeof(ptr);   // returns size of pointer, NOT the array!
}
```

> 🔧 **Portability tip:** Use `sizeof` instead of hardcoding byte counts — sizes vary across platforms.

> ⚡ **Performance tip:** `sizeof` has zero runtime overhead — it is resolved at compile time.

---

## 8. Pointer Arithmetic

### Allowed Operations

| Operation | Example |
|---|---|
| Increment | `++vPtr` or `vPtr++` |
| Decrement | `--vPtr` or `vPtr--` |
| Add integer | `vPtr += 2` |
| Subtract integer | `vPtr -= 3` |
| Subtract pointer from pointer | `n = v2Ptr - vPtr` |

Pointer arithmetic is **only meaningful when operating within the bounds of an array**.

---

### How Pointer Arithmetic Works

When you add an integer `n` to a pointer, the pointer moves forward by `n × sizeof(type)` bytes — not simply by `n`.

**Example:** Given `int v[5]` starting at address 3000 (4-byte integers):

```
v[0]   v[1]   v[2]   v[3]   v[4]
3000   3004   3008   3012   3016
```

```c
int *vPtr = v;       // vPtr = 3000
vPtr += 2;           // vPtr = 3000 + 2×4 = 3008  → points to v[2]
vPtr -= 4;           // vPtr = 3008 - 4×4 = 2992  → DANGER: out of bounds!
```

---

### Subtracting Two Pointers

```c
int v[5];
int *vPtr = &v[0];     // 3000
int *v2Ptr = &v[2];    // 3008

int x = v2Ptr - vPtr;  // x = 2 (number of elements between them, not bytes)
```

> ⚠️ Subtracting pointers that don't point into the **same array** is undefined behavior.

---

### Pointer to `void`

`void *` is a **generic pointer** — it can hold any pointer type without a cast, but it **cannot be dereferenced** directly (the compiler doesn't know the type size):

```c
void *sPtr;
int x = 42;
sPtr = &x;           // OK — any pointer can be assigned to void *
int val = *sPtr;     // ERROR — cannot dereference void *
int val = *((int *) sPtr);  // OK after casting
```

---

### Comparing Pointers

Pointers can be compared with `==`, `!=`, `<`, `>`, etc., but only meaningfully when both point into the **same array**:

```c
if (ptr1 < ptr2)   // ptr1 points to an earlier element than ptr2
if (ptr == NULL)   // most common use: check for null
```

---

## 9. Pointers and Arrays

Arrays and pointers are deeply intertwined in C. An **array name is essentially a constant pointer** to its first element.

```c
int b[5];
int *bPtr = b;       // equivalent to bPtr = &b[0]
```

### Four Equivalent Notations for Array Access

Given `int b[] = {10, 20, 30, 40}` and `int *bPtr = b`:

| Notation | Form | Accesses |
|---|---|---|
| Array index | `b[3]` | element 3 |
| Pointer/offset (array name) | `*(b + 3)` | element 3 |
| Pointer index | `bPtr[3]` | element 3 |
| Pointer/offset (pointer) | `*(bPtr + 3)` | element 3 |

Similarly for addresses:

| `&b[3]` | `b + 3` | `bPtr + 3` |
|---|---|---|
| address of element 3 | same | same |

---

### Array Name is a Constant Pointer

You **cannot** use arithmetic to modify an array name:

```c
b += 3;   // COMPILE ERROR — b is a constant pointer
```

Use a separate pointer variable for traversal instead.

---

### String Copying: Arrays vs. Pointers

**Using array index notation:**
```c
void copy1(char * const s1, const char * const s2) {
    for (size_t i = 0; (s1[i] = s2[i]) != '\0'; ++i) {
        ;   // empty body — all work in the for header
    }
}
```

**Using pointer arithmetic:**
```c
void copy2(char *s1, const char *s2) {
    for (; (*s1 = *s2) != '\0'; ++s1, ++s2) {
        ;   // empty body
    }
}
```

Both do the same thing. The pointer version is idiomatic C — it directly walks through memory without maintaining an index variable.

---

## 10. Arrays of Pointers (String Arrays)

An array can hold pointers. The most common use is an **array of strings**:

```c
const char *suit[4] = {"Hearts", "Diamonds", "Clubs", "Spades"};
```

This does **not** store the strings inside the array. The array holds four **pointers**, each pointing to the first character of a string literal somewhere in memory:

```
suit[0] → 'H' 'e' 'a' 'r' 't' 's' '\0'
suit[1] → 'D' 'i' 'a' 'm' 'o' 'n' 'd' 's' '\0'
suit[2] → 'C' 'l' 'u' 'b' 's' '\0'
suit[3] → 'S' 'p' 'a' 'd' 'e' 's' '\0'
```

**Advantages over a 2D `char` array:**
- No wasted space for padding shorter strings to the length of the longest.
- Strings can be any length.
- Accessing is still simple: `printf("%s", suit[2]);` prints `Clubs`.

---

## 11. Case Study: Card Shuffling and Dealing

This example ties together 2D arrays, arrays of pointers, and random number generation.

### Data Representation

A 4×13 `int` array `deck` represents 52 cards:
- Rows 0–3 → suits (Hearts, Diamonds, Clubs, Spades)
- Columns 0–12 → face values (Ace through King)
- `deck[row][col]` holds the deal-order number (1–52) after shuffling

```c
const char *suit[4] = {"Hearts", "Diamonds", "Clubs", "Spades"};
const char *face[13] = {
    "Ace", "Deuce", "Three", "Four", "Five", "Six", "Seven",
    "Eight", "Nine", "Ten", "Jack", "Queen", "King"
};
unsigned int deck[4][13] = {0};
```

### Shuffling Algorithm

```c
void shuffle(unsigned int wDeck[][13]) {
    for (size_t card = 1; card <= 52; ++card) {
        size_t row, column;
        do {
            row    = rand() % 4;
            column = rand() % 13;
        } while (wDeck[row][column] != 0);   // retry if already placed
        wDeck[row][column] = card;
    }
}
```

> ⚠️ **Performance note:** This algorithm can suffer from **indefinite postponement** — if the random number generator keeps picking occupied slots, the loop may run a very long time. Better algorithms avoid this (e.g., Fisher-Yates shuffle).

### Dealing Algorithm

```c
void deal(unsigned int wDeck[][13], const char *wFace[], const char *wSuit[]) {
    for (size_t card = 1; card <= 52; ++card)
        for (size_t row = 0; row < 4; ++row)
            for (size_t col = 0; col < 13; ++col)
                if (wDeck[row][col] == card)
                    printf("%5s of %-8s%c",
                        wFace[col], wSuit[row],
                        card % 2 == 0 ? '\n' : '\t');
}
```

---

## 12. Pointers to Functions

A **function pointer** holds the starting address of a function in memory — just as an array name holds the address of its first element.

### Declaring a Function Pointer

```c
int (*compare)(int a, int b);
```

Read: *"compare is a pointer to a function that takes two ints and returns an int."*

> ⚠️ Parentheses around `*compare` are **essential**. Without them:
> ```c
> int *compare(int a, int b);   // completely different: a function returning int*
> ```

---

### Passing a Function Pointer to Another Function

```c
void bubble(int work[], size_t size, int (*compare)(int a, int b));
```

**Calling the pointed-to function inside `bubble`:**

```c
if ((*compare)(work[count], work[count + 1]))
    swap(&work[count], &work[count + 1]);
```

Or equivalently (less explicit):

```c
if (compare(work[count], work[count + 1]))
```

---

### Full Example: Ascending/Descending Sort

```c
int ascending(int a, int b)  { return b < a; }   // swap if b < a
int descending(int a, int b) { return b > a; }   // swap if b > a

// In main:
int order;
scanf("%d", &order);

if (order == 1)
    bubble(a, SIZE, ascending);    // pass function pointer
else
    bubble(a, SIZE, descending);
```

The same `bubble` function sorts in **either direction** depending on which comparison function is passed.

---

### Arrays of Function Pointers (Menu-Driven Systems)

```c
void function1(int a) { printf("function1 called with %d\n", a); }
void function2(int b) { printf("function2 called with %d\n", b); }
void function3(int c) { printf("function3 called with %d\n", c); }

// Define array of 3 pointers to functions taking int, returning void:
void (*f[3])(int) = {function1, function2, function3};

// Call based on user input:
size_t choice;
scanf("%u", &choice);
(*f[choice])(choice);   // dereference and call
```

This pattern powers text-based menus, dispatch tables, and callback systems.

---

## 13. Secure C Programming with Pointers

Pointers are a leading source of security vulnerabilities. Key defensive practices:

### Use Safer Standard Library Functions

`printf_s` and `scanf_s` (Annex K of the C standard) check that pointer arguments are non-NULL before use:
- If a `NULL` pointer is passed to `scanf_s`, it returns `EOF`.
- If the format string or a `%s` argument is `NULL` in `printf_s`, output stops and a negative number is returned.

### Key CERT Secure Coding Guidelines

| Guideline | Rule |
|---|---|
| **EXP34-C** | Never dereference a `NULL` pointer — it can crash the program or, in some environments, allow code injection. |
| **DCL13-C** | If a function won't modify what a pointer points to, declare it `const`. |
| **MSC16-C** | Protect function pointers — attackers who overwrite them can redirect execution to malicious code. |

### General Secure Pointer Practices

1. **Always initialize pointers** — to `NULL` if no address is available yet.
2. **Check for `NULL`** before dereferencing any pointer returned from a function (e.g., `malloc`).
3. **Never run off the end of an array** with pointer arithmetic.
4. **Prefer `const`** wherever data shouldn't change — it makes intentions explicit and helps the compiler catch mistakes.
5. **Pass array sizes explicitly** — the array address gives no information about its length.

---

## 14. Common Mistakes Quick Reference

| Mistake | Consequence | Fix |
|---|---|---|
| Uninitialized pointer | Crash or silent corruption | Initialize to `NULL` or an address |
| Dereferencing `NULL` | Crash (segfault) | Check `ptr != NULL` first |
| Dereferencing `void *` | Compile error | Cast to the correct type first |
| `int *x, y;` thinking both are pointers | `y` is an `int` | Use `int *x, *y;` or one per line |
| `sizeof(ptr)` expecting array size | Returns pointer size (4 or 8) | Use `sizeof(array)` in the same scope |
| Pointer arithmetic outside array | Undefined behavior | Only do arithmetic within array bounds |
| Subtracting unrelated pointers | Undefined behavior | Only subtract pointers into the same array |
| Assigning mismatched pointer types | Compiler warning/error | Use `void *` or cast explicitly |
| Modifying an array name with `+=` | Compile error | Use a separate pointer variable |
| Forgetting `&` in `scanf` | Silent data corruption or crash | Always use `scanf("%d", &var)` |

---

## 15. Practice Exercises

### Conceptual Questions

1. What is **indirection**? How does it differ from a direct reference?
2. Why does C require you to pass the size of an array separately when the array is passed to a function?
3. Explain why `sizeof(ptr)` inside a function that receives an array as a parameter does **not** return the size of the array.
4. What is the difference between `const int *p` and `int * const p`?
5. Why is `void *` called a *generic pointer*? What is the one thing you cannot do with it?

---

### Tracing Exercises

**6.** Given:
```c
int a[5] = {10, 20, 30, 40, 50};
int *p = a;
```
What does each of the following evaluate to?

| Expression | Value |
|---|---|
| `*p` | |
| `*(p + 3)` | |
| `p[2]` | |
| `*(a + 4)` | |
| `p + 2 == &a[2]` | |

---

**7.** Trace this code and write the output:
```c
int x = 5;
int *p = &x;
*p = *p * *p;
printf("%d %d\n", x, *p);
```

---

### Coding Exercises

**8.** Write a function `swap(int *a, int *b)` that exchanges two integers. Write a `main` that tests it.

**9.** Write a function `reverseArray(int *arr, size_t n)` that reverses an integer array in place using pointer arithmetic (no index variables allowed).

**10.** Write a function `stringLength(const char *s)` that returns the length of a string using pointer arithmetic (do not use `strlen` or array indexing).

**11.** Write a program that:
- Defines an array of 5 function pointers, each pointing to a function that takes an `int` and returns its square, cube, double, half (integer division), and negative.
- Prompts the user to enter a number and an operation index (0–4), then calls the appropriate function and prints the result.

**12.** *(Challenge)* Implement the Fisher-Yates shuffle on the card deck from Section 11. This algorithm guarantees each card is placed exactly once, eliminating the indefinite-postponement problem of the random-retry approach.

---

### Find-the-Bug Exercises

**13.** Find all errors:
```c
int z[5] = {1, 2, 3, 4, 5};
int *zPtr;

++zPtr;              // (a)
int n = zPtr;        // (b)
int m = *zPtr[2];    // (c)
++z;                 // (d)
```

**14.** What is wrong with this function, and what are the consequences?
```c
void doubleAll(int *arr, size_t n) {
    for (size_t i = 0; i <= n; ++i)   // note: <=
        arr[i] *= 2;
}
```

---

*Good luck! Mastering pointers is the turning point in becoming a confident C programmer.*
