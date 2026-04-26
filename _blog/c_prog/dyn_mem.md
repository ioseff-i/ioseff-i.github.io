---
title: "Dynamic Memory Management"
layout: single
author_profile: true
mathjax: true
---

> **Course Level:** Undergraduate CS / Systems Programming  
> **Prerequisites:** C basics, pointers, arrays (see: *C Pointers: Zero to Mastery*)  
> **What you will master:** Stack vs. heap, why static allocation falls short, `malloc` / `calloc` / `realloc` / `free`, safe patterns, dynamic 1D and 2D arrays.

---

## Table of Contents

1. [How Memory is Organized in a Running Program](#1-how-memory-is-organized-in-a-running-program)
2. [The Stack — What It Gives You For Free (and What It Takes Away)](#2-the-stack)
3. [Variable-Length Arrays — A Tempting Shortcut with a Hidden Cost](#3-variable-length-arrays-vlas)
4. [Why Functions Cannot Safely Return Pointers to Local Variables](#4-why-functions-cannot-safely-return-pointers-to-local-variables)
5. [Why We Need Dynamic Memory — The Motivating Problem](#5-why-we-need-dynamic-memory)
6. [The Heap — Memory You Control](#6-the-heap)
7. [`malloc` — Allocating a Raw Block](#7-malloc)
8. [`calloc` — Allocating and Zeroing](#8-calloc)
9. [`free` — Giving Memory Back](#9-free)
10. [Handling `NULL` — What to Do When Allocation Fails](#10-handling-null)
11. [`realloc` — Resizing a Block Safely](#11-realloc)
12. [The Safe `realloc` Pattern — Never Lose Your Original Pointer](#12-the-safe-realloc-pattern)
13. [The Elegant Way to Free — A Clean-Up Discipline](#13-the-elegant-way-to-free)
14. [Dynamic 1D Arrays — Full Working Example](#14-dynamic-1d-arrays)
15. [Dynamic 2D Arrays — Three Approaches](#15-dynamic-2d-arrays)
16. [Common Mistakes Quick Reference](#16-common-mistakes-quick-reference)
17. [Practice Exercises](#17-practice-exercises)

---

## 1. How Memory is Organized in a Running Program

Before touching a single allocation function, you need a mental map of where data lives. Every running C program sees its memory divided into distinct regions:

```
High addresses
┌──────────────────────────────┐
│         Stack                │  ← grows downward
│  (local variables, frames)   │
├──────────────────────────────┤
│             ↓                │
│         (free space)         │
│             ↑                │
├──────────────────────────────┤
│          Heap                │  ← grows upward
│  (dynamic allocations)       │
├──────────────────────────────┤
│    BSS / Data segments       │  ← global & static variables
├──────────────────────────────┤
│         Text segment         │  ← compiled code (read-only)
└──────────────────────────────┘
Low addresses
```

| Region | What lives there | Lifetime | Who manages it |
|---|---|---|---|
| **Text** | Compiled machine code | Duration of program | OS |
| **Data / BSS** | Global and `static` variables | Duration of program | Compiler |
| **Stack** | Local variables, function parameters, return addresses | One function call | Compiler / CPU |
| **Heap** | Dynamically allocated memory | Until explicitly freed | **You** |

The crucial insight: **stack memory lives and dies with the function that created it. Heap memory lives until you free it.**

---

## 2. The Stack

### What the Stack Gives You

Every time you call a function, the CPU automatically carves out a **stack frame** — a chunk of the stack — to hold:
- The function's local variables
- Its parameters
- The return address (where to go when the function ends)

You get all of this **for free, automatically**:

```c
void compute(int n) {
    int result = n * n;   // allocated on the stack automatically
    double buffer[100];   // also on the stack — 800 bytes, instantly
    // ...
}   // stack frame is destroyed here — result and buffer are gone
```

This is fast, simple, and requires zero cleanup code.

### What the Stack Takes Away

The stack has two hard limitations:

**1. Fixed, limited size.**  
The stack is typically 1–8 MB. Declare a large array as a local variable and you overflow it — causing a **stack overflow crash**:

```c
void dangerous(void) {
    int huge[10000000];   // ~40 MB on the stack → immediate crash
}
```

**2. Memory doesn't outlive the function.**  
Once a function returns, its entire stack frame is gone. Any pointer into that frame now points to memory that has been reused by the next function call — a **dangling pointer**:

```c
int *danger(void) {
    int x = 42;
    return &x;   // x lives on the stack — it dies when danger() returns
}

int main(void) {
    int *p = danger();
    printf("%d\n", *p);   // UNDEFINED BEHAVIOR — p points to dead memory
}
```

This is so important it gets its own section.

---

## 3. Variable-Length Arrays (VLAs)

### What They Are

C99 added **variable-length arrays** — arrays whose size is determined at runtime rather than compile time:

```c
void process(int n) {
    int arr[n];    // size is not a compile-time constant — this is a VLA
    // ...
}
```

Before VLAs, you had to either use a fixed worst-case size or use `malloc`. VLAs look like a clean middle ground.

### Why VLAs Are Problematic

Despite their convenience, VLAs come with serious drawbacks:

**1. Still on the stack.**  
A VLA is allocated on the stack, just like any other local array. If `n` is large, you still get a stack overflow — except now you don't even see a suspiciously large constant in the source code:

```c
void risky(int n) {
    int arr[n];    // if n = 1,000,000 → crash
                   // the compiler cannot warn you — n is only known at runtime
}
```

**2. No way to check if allocation succeeded.**  
`malloc` returns `NULL` on failure. A VLA that overflows the stack doesn't return anything — the program simply crashes with no recovery path.

**3. Made optional in C11.**  
The C11 standard made VLA support optional for compilers. Code using VLAs may not compile on all platforms or embedded targets.

**4. Banned in many professional codebases.**  
The Linux kernel, MISRA C (automotive/aerospace), and many security-sensitive codebases **prohibit VLAs** precisely because of the uncontrollable stack risk.

### When VLAs Are Acceptable

Small, provably-bounded sizes where a stack overflow is physically impossible — for example, `int row[ncols]` when `ncols` is known to be at most 10. Even then, a fixed-size array is usually clearer.

**The rule of thumb:** If the size comes from user input, a file, or any external source — use the heap, not a VLA.

---

## 4. Why Functions Cannot Safely Return Pointers to Local Variables

This is one of the most important rules in C. It deserves a thorough explanation.

### The Stack Frame Lifecycle

```c
int *broken(void) {
    int result = 100;
    return &result;      // address of a stack variable
}                        // stack frame is destroyed HERE

int main(void) {
    int *p = broken();   // p holds an address that no longer means anything
    printf("%d\n", *p);  // reads garbage — or crashes
}
```

Here is what happens in memory, step by step:

```
Step 1: main() calls broken()
        Stack: [ main's frame | broken's frame: result=100 ]

Step 2: broken() returns &result → address 0xABCD (example)

Step 3: broken()'s frame is DESTROYED
        Stack: [ main's frame | ??? ] ← 0xABCD is now "free" stack space

Step 4: main() stores p = 0xABCD

Step 5: Next function call (even printf itself!) reuses that stack space
        *p now reads whatever printf put there — not 100
```

The compiler will often warn you:

```
warning: function returns address of local variable [-Wreturn-local-addr]
```

**Always heed this warning. It is never safe to ignore it.**

### What You CAN Return from a Function

| What you return | Safe? | Why |
|---|---|---|
| A value (int, double, struct…) | ✅ | Value is copied to caller |
| Pointer to a `static` local variable | ⚠️ | Safe, but not thread-safe |
| Pointer to a global variable | ✅ | Lives for program duration |
| Pointer to heap memory (`malloc`) | ✅ | Lives until `free` is called |
| Pointer to a local (stack) variable | ❌ | Dangling pointer — undefined behavior |

**The correct solution for returning an array or a struct from a function is to allocate it on the heap.**

---

## 5. Why We Need Dynamic Memory — The Motivating Problem

Let's make the need concrete.

### Problem 1: Size Unknown at Compile Time

```c
// How many students? We don't know until the user tells us.
int n;
scanf("%d", &n);

// We cannot do this safely:
int grades[n];      // VLA — stack risk, no error checking

// We need this:
int *grades = malloc(n * sizeof(int));  // heap — safe, checkable
```

### Problem 2: Data That Must Outlive the Function That Creates It

```c
// Trying to build a list of names in a helper function:
char **buildNameList(int count) {
    char *names[count];   // local VLA of pointers
    // ... fill names ...
    return names;          // WRONG: returns address of stack memory
}
```

With dynamic memory, the function can create data on the heap and return a valid pointer to it.

### Problem 3: Growing and Shrinking Collections

You're reading records from a file but don't know how many there are. You start with space for 10, and if you hit 11 records, you need more space. **This is impossible with static or stack arrays.** `realloc` on a heap allocation handles it cleanly.

---

## 6. The Heap

The heap is a large pool of memory managed by the C runtime library. Unlike the stack:

- You request memory explicitly with `malloc`, `calloc`, or `realloc`.
- Memory stays allocated until you release it with `free`.
- Allocations can be any size (up to available system memory).
- There is no automatic cleanup — if you forget to `free`, that memory is **leaked**.

The four functions you need are all in `<stdlib.h>`:

```c
#include <stdlib.h>
```

---

## 7. `malloc` — Allocating a Raw Block

### Signature

```c
void *malloc(size_t size);
```

- Requests `size` bytes of memory from the heap.
- Returns a `void *` (generic pointer) to the first byte of the block.
- Returns `NULL` if the allocation fails (not enough memory).
- **The memory is uninitialized — it contains garbage values.**

### Basic Usage

```c
// Allocate space for one int:
int *p = malloc(sizeof(int));

// Allocate space for an array of n doubles:
int n = 50;
double *arr = malloc(n * sizeof(double));
```

### Why `sizeof` in Every `malloc`

Always use `sizeof(type)` rather than a hardcoded byte count:

```c
// FRAGILE — assumes int is 4 bytes:
int *p = malloc(4);

// CORRECT — works on any platform:
int *p = malloc(sizeof(int));

// BEST for arrays — explicit and readable:
int *arr = malloc(n * sizeof(*arr));  // sizeof(*arr) = sizeof(int)
```

The `sizeof(*arr)` form is particularly robust: if you later change `arr`'s type, the `sizeof` updates automatically.

### The Return Type is `void *`

`malloc` returns `void *` because it doesn't know what type you need. In C, `void *` is **automatically converted** to any pointer type without a cast:

```c
int    *ip = malloc(sizeof(int));       // no cast needed in C
double *dp = malloc(sizeof(double));    // no cast needed in C
```

> ⚠️ In C++ you **do** need an explicit cast. In C you do not — and adding unnecessary casts can actually hide bugs (like forgetting to `#include <stdlib.h>`).

---

## 8. `calloc` — Allocating and Zeroing

### Signature

```c
void *calloc(size_t nmemb, size_t size);
```

- Allocates space for `nmemb` elements of `size` bytes each.
- **Initializes every byte to zero.**
- Returns `NULL` on failure.

### `calloc` vs. `malloc`

```c
// malloc — fast, but garbage values:
int *a = malloc(100 * sizeof(int));
// a[0], a[1], ... contain random garbage

// calloc — slightly slower, but all zeros:
int *b = calloc(100, sizeof(int));
// b[0] == 0, b[1] == 0, ..., b[99] == 0
```

### When to Choose Each

| Situation | Use |
|---|---|
| You will immediately write to every element | `malloc` (faster) |
| You need zero-initialization (counters, flags, matrices) | `calloc` |
| You're allocating a struct and want all fields zeroed | `calloc` |
| Performance is critical and you handle init yourself | `malloc` |

```c
// Practical example: frequency counter
int *freq = calloc(256, sizeof(int));   // all counts start at 0
// Now count characters in a string:
for (char *c = text; *c != '\0'; ++c)
    freq[(unsigned char)*c]++;
```

---

## 9. `free` — Giving Memory Back

### Signature

```c
void free(void *ptr);
```

Every `malloc` or `calloc` must eventually be matched with exactly one `free`.

```c
int *arr = malloc(10 * sizeof(int));
// ... use arr ...
free(arr);   // return memory to the heap
```

### Rules of `free`

**Rule 1: Only free heap memory.**  
Do not `free` a pointer to a stack variable or a string literal.

```c
int x = 5;
free(&x);            // UNDEFINED BEHAVIOR — x is on the stack

char *s = "hello";
free(s);             // UNDEFINED BEHAVIOR — string literal is in text segment
```

**Rule 2: Free exactly once.**  
Freeing a pointer twice (**double free**) corrupts the heap and is a serious security vulnerability:

```c
free(arr);
free(arr);   // DOUBLE FREE — undefined behavior, heap corruption
```

**Rule 3: `free(NULL)` is always safe.**  
The standard guarantees that `free(NULL)` does nothing. This is very useful for defensive programming:

```c
int *p = NULL;
free(p);   // perfectly safe — does nothing
```

**Rule 4: After freeing, set the pointer to `NULL`.**  
This turns a dangling pointer into a safely checkable NULL pointer — preventing accidental use-after-free:

```c
free(arr);
arr = NULL;   // now any accidental dereference → crash immediately (detectable)
              // rather than: silent data corruption (undetectable)
```

---

## 10. Handling `NULL` — What to Do When Allocation Fails

`malloc` and `calloc` return `NULL` when the system cannot satisfy the request. **You must always check for this.** Dereferencing a `NULL` pointer crashes the program or worse.

### The Basic Check

```c
int *arr = malloc(n * sizeof(int));
if (arr == NULL) {
    fprintf(stderr, "Error: memory allocation failed for %zu elements\n", n);
    exit(EXIT_FAILURE);
}
```

### Why `stderr` and `exit`?

- `stderr` is unbuffered — your error message appears immediately even if `stdout` hasn't flushed.
- `exit(EXIT_FAILURE)` cleanly terminates the program with a non-zero status code so scripts can detect the failure.

### A Helper Function Pattern

For larger programs, define a checked allocation wrapper so you never forget the check:

```c
#include <stdlib.h>
#include <stdio.h>

void *safeMalloc(size_t size) {
    void *p = malloc(size);
    if (p == NULL) {
        fprintf(stderr, "Fatal: malloc(%zu) failed\n", size);
        exit(EXIT_FAILURE);
    }
    return p;
}

void *safeCalloc(size_t nmemb, size_t size) {
    void *p = calloc(nmemb, size);
    if (p == NULL) {
        fprintf(stderr, "Fatal: calloc(%zu, %zu) failed\n", nmemb, size);
        exit(EXIT_FAILURE);
    }
    return p;
}
```

Now every allocation automatically handles failure:

```c
int *arr = safeMalloc(n * sizeof(int));   // guaranteed non-NULL
```

---

## 11. `realloc` — Resizing a Block

### Signature

```c
void *realloc(void *ptr, size_t newSize);
```

Resizes a previously allocated block to `newSize` bytes.

**What `realloc` does internally:**

1. If the block can be expanded in place (there's free space right after it on the heap), it does so and returns the **same pointer**.
2. If it cannot expand in place, it:
   - Allocates a new block of `newSize` bytes elsewhere on the heap.
   - Copies the old data into the new block.
   - Frees the old block.
   - Returns a **new pointer**.
3. If allocation fails, returns `NULL` — and leaves the **original block untouched**.

### Basic Usage

```c
int capacity = 10;
int *arr = malloc(capacity * sizeof(int));

// ... fill arr with data, then need more space ...
capacity = 20;
arr = realloc(arr, capacity * sizeof(int));  // DANGEROUS — see next section!
```

The line above works most of the time — but contains a critical bug. Let's fix it properly.

---

## 12. The Safe `realloc` Pattern — Never Lose Your Original Pointer

### The Bug

```c
// DANGEROUS PATTERN:
arr = realloc(arr, newCapacity * sizeof(int));
```

If `realloc` returns `NULL` (allocation failed), you have just overwritten `arr` with `NULL`. The original memory block is now **unreachable** — you have a **memory leak** and have **lost all your data**.

### The Safe Pattern: Always Use a Temporary Variable

```c
// SAFE PATTERN:
int *temp = realloc(arr, newCapacity * sizeof(int));
if (temp == NULL) {
    // realloc failed — arr still points to the original data
    fprintf(stderr, "Error: realloc failed\n");
    free(arr);        // clean up the original block
    arr = NULL;
    exit(EXIT_FAILURE);
}
arr = temp;           // only update arr after confirming success
```

**Why this works:**
- `temp` receives the return value of `realloc`.
- If `temp == NULL`, `arr` is still valid — your data is safe.
- You can decide to handle the failure gracefully (maybe reduce the request, or report the error and free cleanly).
- Only when `realloc` succeeds do you update `arr`.

### Memory Layout During a Successful `realloc`

```
Before realloc (capacity = 4):
Heap: [ 10 | 20 | 30 | 40 ]
       ↑
       arr (address 0x1000)

After realloc (capacity = 8) — expanded in place:
Heap: [ 10 | 20 | 30 | 40 | ?? | ?? | ?? | ?? ]
       ↑
       arr (same address 0x1000 — returned as temp)

After realloc (capacity = 8) — moved to new location:
Heap: [ 10 | 20 | 30 | 40 | ?? | ?? | ?? | ?? ]  (at 0x2000)
       old block at 0x1000 is now freed by realloc
       ↑
       arr = temp (new address 0x2000)
```

---

## 13. The Elegant Way to Free — A Clean-Up Discipline

Memory management errors (leaks, double frees, use-after-free) are far easier to prevent with a consistent discipline. Here is the cleanest approach:

### The Three-Step Free Pattern

```c
free(ptr);
ptr = NULL;
```

Always in this order, always together. Never leave a freed pointer pointing at dead memory.

### Centralize Clean-Up with `goto` (or a cleanup function)

For functions that allocate multiple resources, use a single exit path:

```c
int processData(int n) {
    int *data    = malloc(n * sizeof(int));
    double *temp = malloc(n * sizeof(double));
    char *buf    = malloc(256);

    if (!data || !temp || !buf) {
        goto cleanup;   // jump to centralized cleanup
    }

    // ... do real work ...

cleanup:
    free(data);   data = NULL;
    free(temp);   temp = NULL;
    free(buf);    buf  = NULL;
    // free(NULL) is safe — no double-free risk even if malloc failed
    return 0;
}
```

This pattern ensures every allocation is freed regardless of which step failed, and `free(NULL)` being safe means you don't need to check which ones succeeded.

### Writing a Dedicated Free Function

For complex types (like dynamic 2D arrays), write a matching free function alongside your allocation function:

```c
double **createMatrix(int rows, int cols);
void    freeMatrix(double **matrix, int rows);
```

Paired allocation/free functions are much easier to audit than scattered `free` calls throughout the codebase.

### Memory Leak Detection

Even with discipline, leaks happen. Use tools to catch them:

- **Valgrind** (Linux/macOS): `valgrind --leak-check=full ./your_program`
- **AddressSanitizer** (gcc/clang): compile with `-fsanitize=address`

Both tools report exactly which allocation was leaked and where it was made.

---

## 14. Dynamic 1D Arrays — Full Working Example

Let's build a complete, production-quality dynamic array that can grow as needed.

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    size_t capacity = 4;    // initial capacity
    size_t count    = 0;    // current number of elements

    // --- Allocate initial array ---
    int *arr = malloc(capacity * sizeof(int));
    if (arr == NULL) {
        fprintf(stderr, "Initial allocation failed\n");
        return EXIT_FAILURE;
    }

    // --- Simulate reading values until -1 ---
    int value;
    printf("Enter integers (-1 to stop):\n");

    while (scanf("%d", &value) == 1 && value != -1) {

        // If full, double the capacity
        if (count == capacity) {
            size_t newCapacity = capacity * 2;
            int *temp = realloc(arr, newCapacity * sizeof(int));

            if (temp == NULL) {
                fprintf(stderr, "realloc failed at capacity %zu\n", newCapacity);
                free(arr);
                arr = NULL;
                return EXIT_FAILURE;
            }

            arr      = temp;
            capacity = newCapacity;
            printf("  [array grew to capacity %zu]\n", capacity);
        }

        arr[count++] = value;
    }

    // --- Print contents ---
    printf("\nStored %zu values:\n", count);
    for (size_t i = 0; i < count; ++i)
        printf("  arr[%zu] = %d\n", i, arr[i]);

    // --- Optional: shrink to exact size to reclaim unused space ---
    if (count > 0 && count < capacity) {
        int *temp = realloc(arr, count * sizeof(int));
        if (temp != NULL) {           // shrinking rarely fails, but check anyway
            arr      = temp;
            capacity = count;
        }
    }

    // --- Free ---
    free(arr);
    arr = NULL;

    return EXIT_SUCCESS;
}
```

**Key behaviors demonstrated:**
- Always check the initial `malloc`.
- Use a `capacity` / `count` pair — never let `count` reach `capacity` without growing first.
- Double the capacity on each growth (amortized O(1) insertions, same strategy as C++ `std::vector`).
- Use the safe `temp` pattern with `realloc`.
- Set `arr = NULL` after freeing.

---

## 15. Dynamic 2D Arrays — Three Approaches

2D arrays are where dynamic memory gets genuinely tricky. There are three approaches, each with different trade-offs.

---

### Approach 1: Array of Pointers (Jagged / Non-Contiguous)

Allocate an array of row pointers, then allocate each row separately.

```c
#include <stdio.h>
#include <stdlib.h>

// Create an rows × cols matrix
double **createMatrix(int rows, int cols) {
    // Step 1: allocate the array of row pointers
    double **matrix = malloc(rows * sizeof(double *));
    if (matrix == NULL) return NULL;

    // Step 2: allocate each row
    for (int i = 0; i < rows; ++i) {
        matrix[i] = malloc(cols * sizeof(double));
        if (matrix[i] == NULL) {
            // Clean up already-allocated rows before returning NULL
            for (int j = 0; j < i; ++j)
                free(matrix[j]);
            free(matrix);
            return NULL;
        }
    }
    return matrix;
}

// Free the matrix — must know number of rows
void freeMatrix(double **matrix, int rows) {
    if (matrix == NULL) return;
    for (int i = 0; i < rows; ++i) {
        free(matrix[i]);
        matrix[i] = NULL;
    }
    free(matrix);
    // caller should set their pointer to NULL
}

int main(void) {
    int rows = 3, cols = 4;

    double **m = createMatrix(rows, cols);
    if (m == NULL) {
        fprintf(stderr, "Matrix allocation failed\n");
        return EXIT_FAILURE;
    }

    // Use exactly like a 2D array:
    for (int i = 0; i < rows; ++i)
        for (int j = 0; j < cols; ++j)
            m[i][j] = i * cols + j;

    // Print:
    for (int i = 0; i < rows; ++i) {
        for (int j = 0; j < cols; ++j)
            printf("%6.1f ", m[i][j]);
        printf("\n");
    }

    freeMatrix(m, rows);
    m = NULL;
    return EXIT_SUCCESS;
}
```

**Memory layout:**

```
m (pointer to pointer array)
│
├─ m[0] ──→ [ 0.0 | 1.0 | 2.0 | 3.0 ]   (row 0 — separate heap block)
├─ m[1] ──→ [ 4.0 | 5.0 | 6.0 | 7.0 ]   (row 1 — separate heap block)
└─ m[2] ──→ [ 8.0 | 9.0 | 10.0| 11.0]   (row 2 — separate heap block)
```

**Trade-offs:**

| | |
|---|---|
| ✅ Natural `m[i][j]` syntax | |
| ✅ Rows can be different lengths (jagged arrays) | |
| ✅ Individual rows can be reallocated | |
| ❌ `rows + 1` separate `malloc` calls | |
| ❌ Rows are scattered in memory — poor cache performance | |
| ❌ Freeing requires a loop | |

---

### Approach 2: Single Contiguous Block + Index Arithmetic

Allocate all `rows × cols` elements in one block, then compute the element address manually.

```c
int rows = 3, cols = 4;

// One allocation for all data:
double *matrix = malloc(rows * cols * sizeof(double));
if (matrix == NULL) { /* handle error */ }

// Access element [i][j]:
// matrix[i * cols + j]

for (int i = 0; i < rows; ++i)
    for (int j = 0; j < cols; ++j)
        matrix[i * cols + j] = i * cols + j;

// Print:
for (int i = 0; i < rows; ++i) {
    for (int j = 0; j < cols; ++j)
        printf("%6.1f ", matrix[i * cols + j]);
    printf("\n");
}

// Free — single call:
free(matrix);
matrix = NULL;
```

**Memory layout:**

```
matrix ──→ [ 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 ]
             ↑─────row 0──────↑──────row 1──────↑──────row 2──────↑
```

**Trade-offs:**

| | |
|---|---|
| ✅ Single `malloc` — simple error handling | |
| ✅ All data contiguous — excellent cache performance | |
| ✅ Single `free` call | |
| ❌ Cannot use `m[i][j]` syntax — must write `m[i * cols + j]` | |
| ❌ Rows must all be the same length | |

---

### Approach 3: Pointer Array + Contiguous Data (Best of Both Worlds)

Combine the two approaches: one allocation for all data, and a separate array of row pointers that point into it.

```c
double **createContiguousMatrix(int rows, int cols) {
    // Step 1: allocate the data block
    double *data = malloc(rows * cols * sizeof(double));
    if (data == NULL) return NULL;

    // Step 2: allocate the array of row pointers
    double **matrix = malloc(rows * sizeof(double *));
    if (matrix == NULL) {
        free(data);
        return NULL;
    }

    // Step 3: point each row pointer into the data block
    for (int i = 0; i < rows; ++i)
        matrix[i] = data + i * cols;   // pointer arithmetic

    return matrix;
}

void freeContiguousMatrix(double **matrix) {
    if (matrix == NULL) return;
    free(matrix[0]);   // frees the entire data block (matrix[0] points to it)
    free(matrix);      // frees the pointer array
}

// Usage:
double **m = createContiguousMatrix(3, 4);
m[1][2] = 99.5;   // natural syntax ✓
freeContiguousMatrix(m);
m = NULL;
```

**Memory layout:**

```
matrix (pointer array — one block)
│
├─ matrix[0] ──→ data[0]  data[1]  data[2]  data[3]   ← row 0 │
├─ matrix[1] ──→ data[4]  data[5]  data[6]  data[7]   ← row 1 │ contiguous
└─ matrix[2] ──→ data[8]  data[9]  data[10] data[11]  ← row 2 │
```

**Trade-offs:**

| | |
|---|---|
| ✅ Natural `m[i][j]` syntax | |
| ✅ All data contiguous — cache-friendly | |
| ✅ Only two `malloc` calls regardless of number of rows | |
| ✅ Only two `free` calls | |
| ❌ Slightly more complex setup code | |
| ❌ Rows must all be the same length | |

### Choosing the Right Approach

| Need | Approach |
|---|---|
| Maximum simplicity of allocation/free | 2 (flat array, index arithmetic) |
| Natural `m[i][j]` syntax + best performance | 3 (contiguous + pointer array) |
| Rows of different lengths (jagged array) | 1 (array of independent row pointers) |
| Resizing individual rows independently | 1 |

---

## 16. Common Mistakes Quick Reference

| Mistake | What Goes Wrong | Fix |
|---|---|---|
| Using an uninitialized pointer | Crash or data corruption | Initialize to `NULL` or an address immediately |
| Not checking `malloc`/`calloc` for `NULL` | Null-pointer dereference → crash | Always check: `if (ptr == NULL) { ... }` |
| `arr = realloc(arr, ...)` directly | Memory leak if `realloc` fails | Use a temp: `temp = realloc(arr, ...); if (temp) arr = temp;` |
| Forgetting `free` | Memory leak | Every `malloc` needs exactly one `free` |
| Double `free` | Heap corruption, security vulnerability | Set pointer to `NULL` after freeing |
| Using memory after `free` | Reads garbage or crashes | Set pointer to `NULL` immediately after `free` |
| Returning pointer to local variable | Dangling pointer | Return heap-allocated memory or a value |
| VLA with user-provided size | Stack overflow crash | Use `malloc` for any externally-provided size |
| `free`-ing a stack variable | Undefined behavior | Only `free` what was heap-allocated |
| `free`-ing part of an array | Heap corruption | Only `free` the original pointer from `malloc` |
| Forgetting to free each row in a jagged 2D array | Leak of row data | Loop and `free` each row, then `free` the pointer array |
| `sizeof(ptr)` where you want array size | Gets pointer size (4 or 8), not array size | Pass size separately; use `sizeof` only where the array is defined |

---

## 17. Practice Exercises

### Conceptual Questions

1. Explain the difference between the stack and the heap in terms of: who manages the memory, how long it lasts, and what can go wrong.

2. Why is it dangerous to return a pointer to a local variable from a function? What should you return instead if the caller needs the data to persist?

3. What is the difference between `malloc` and `calloc`? Give a scenario where `calloc` is the better choice.

4. Why should you use a temporary pointer variable when calling `realloc` instead of assigning the result directly back to your original pointer?

5. Why is `free(NULL)` safe, and how does this property simplify error-handling cleanup code?

6. A student declares `int arr[n]` inside a function (VLA) where `n` comes from `scanf`. List three things that could go wrong that would not happen with a `malloc`-based solution.

---

### Tracing Exercises

**7.** What is the output, and what undefined behavior (if any) exists?

```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
printf("%d\n", *p);   // (a)
free(p);              // (b)
```

**8.** Trace the memory state after each line:

```c
int *arr = malloc(3 * sizeof(int));  // (a)
arr[0] = 10; arr[1] = 20; arr[2] = 30;
int *temp = realloc(arr, 5 * sizeof(int));  // (b) — assume succeeds
arr = temp;  // (c)
arr[3] = 40; arr[4] = 50;
free(arr); arr = NULL;  // (d)
```

**9.** Find every memory management error in this code:

```c
int *data = malloc(10 * sizeof(int));
for (int i = 0; i <= 10; i++)    // (a)
    data[i] = i;
data = realloc(data, 20 * sizeof(int));  // (b)
printf("%d\n", data[5]);
int x = 5;
free(&x);   // (c)
free(data);
free(data); // (d)
```

---

### Coding Exercises

**10.** Write a function `int *readInts(size_t *outCount)` that reads integers from standard input until EOF, storing them in a dynamically growing array (starting with capacity 4, doubling when full). Return the array and set `*outCount` to the number of elements read. The caller is responsible for freeing the returned pointer.

**11.** Write `char *duplicateString(const char *s)` that allocates exactly enough heap memory to hold a copy of `s` (including the null terminator), copies the string in, and returns the pointer. Handle allocation failure by returning `NULL`.

**12.** Write `double **identityMatrix(int n)` that creates an `n × n` identity matrix on the heap using Approach 3 (contiguous data + pointer array). Write the matching `freeIdentityMatrix(double **m)`. Test it for `n = 4`.

**13.** *(Challenge)* Implement a simple dynamic array (like a `struct`) with the following interface:

```c
typedef struct {
    int    *data;
    size_t  count;
    size_t  capacity;
} IntArray;

IntArray intArrayCreate(void);           // creates empty array
void     intArrayPush(IntArray *a, int value);  // appends, grows if needed
int      intArrayGet(IntArray *a, size_t index);
void     intArrayFree(IntArray *a);      // frees and zeroes all fields
```

Use the safe `realloc` pattern inside `intArrayPush`. Demonstrate it with a `main` that pushes 100 values and prints them.

**14.** *(Challenge)* Write a function that reads a 2D matrix of doubles from a file where the first line contains `rows cols`, and the remaining lines contain the data. Use Approach 2 (flat array). Return `NULL` on any error. Write the matching `main` that prints the matrix and frees it.

---

*Once you can write, grow, reshape, and free heap memory without leaks or corruption, you have the foundation for every advanced C data structure: linked lists, trees, hash tables, and beyond.*
