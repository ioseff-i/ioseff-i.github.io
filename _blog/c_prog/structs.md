---
title: " Enumerated, Structure, and Union Types in C"
layout: single
author_profile: true
mathjax: true
---

## Table of Contents

1. [Derived Types — The Big Picture](#1-derived-types--the-big-picture)
2. [The Type Definition (`typedef`)](#2-the-type-definition-typedef)
3. [Enumerated Types (`enum`)](#3-enumerated-types-enum)
4. [Structures (`struct`)](#4-structures-struct)
   - 4.1 [Declaring a Structure](#41-declaring-a-structure)
   - 4.2 [Initializing a Structure](#42-initializing-a-structure)
   - 4.3 [Accessing Fields — The Dot Operator](#43-accessing-fields--the-dot-operator)
   - 4.4 [Copying a Structure](#44-copying-a-structure)
   - 4.5 [Pointers to Structures — The Arrow Operator](#45-pointers-to-structures--the-arrow-operator)
   - 4.6 [Arrays Inside Structures](#46-arrays-inside-structures)
   - 4.7 [Arrays of Structures](#47-arrays-of-structures)
   - 4.8 [Nested Structures](#48-nested-structures)
   - 4.9 [Structure Padding](#49-structure-padding)
   - 4.10 [Structures and Functions](#410-structures-and-functions)
5. [Union Types (`union`)](#5-union-types-union)
6. [Putting It All Together — Complete Programs](#6-putting-it-all-together--complete-programs)
7. [Guided Exercises (with Solutions)](#7-guided-exercises-with-solutions)
8. [Quick-Reference Cheat Sheet](#8-quick-reference-cheat-sheet)

---

## 1. Derived Types — The Big Picture

In C, **derived types** are types built from simpler ones. They include:

| Derived Type | Where Introduced |
|---|---|
| Function Type | Ch. 4 |
| Array Type | Ch. 8 |
| Pointer Type | Ch. 9 |
| **Structure Type** | **This tutorial** |
| **Union Type** | **This tutorial** |
| **Enumerated Type** | **This tutorial** |

Think of standard types (`int`, `float`, `char`, …) as raw ingredients and derived types as the dishes you cook from them.

---

## 2. The Type Definition (`typedef`)

### What is it?

`typedef` gives an **alias** (a second name) to any existing type. It does **not** create a new type; it simply provides a more readable or portable name for one that already exists.

### Syntax

```
typedef  existing_type  NEW_IDENTIFIER;
```

By convention the new identifier is written in **UPPERCASE** (though this is style, not a rule).

### Simple Example

```c
typedef int frog;   // "frog" is now an alias for int

int  a;   // ordinary int
frog b;   // also an int — the two are identical
```

Both `a` and `b` occupy the same number of bytes and behave the same way.

### Why is `typedef` Useful?

**Reason 1 — Readability.** Structure declarations are verbose:

```c
struct Point { int x; int y; };

// Without typedef — you must write "struct Point" everywhere:
struct Point p1;

// With typedef — much cleaner:
typedef struct { int x; int y; } Point;
Point p1;
```

**Reason 2 — Portability.** If you need a guaranteed 32-bit integer:

```c
typedef int       Int32;   /* 4-byte variable on this platform */
typedef short int Int16;   /* 2-byte variable on this platform */
```

When you port the code to a different architecture, you only change the two `typedef` lines — all the code using `Int32` adapts automatically.

---

## 3. Enumerated Types (`enum`)

### Concept

An **enumerated type** is a user-defined integer type where each possible integer value is given a descriptive name called an **enumeration constant**.

> **Mental model:** an enum lets you replace magic numbers with meaningful names.

```c
// Without enum — confusing:
int day = 3;

// With enum — self-documenting:
enum Day { MON=1, TUE, WED, THU, FRI, SAT, SUN };
enum Day today = WED;   // today == 3
```

### Declaring an Enum

```c
enum TagName { CONST1, CONST2, CONST3, ... };
```

By default the compiler assigns consecutive integer values starting at **0**:

| Constant | Default Value |
|---|---|
| `CONST1` | 0 |
| `CONST2` | 1 |
| `CONST3` | 2 |

### Custom Values

You can override any value; subsequent constants continue from that new value:

```c
enum TV {
    FOX  = 2,
    NBC  = 4,
    CBS  = 5,
    ABC  = 11,
    HBO  = 15,
    SHOW = 17,
    MAX  = 31,
    ESPN = 39,
    CNN  = 51
};
```

### Months Example

```c
#include <stdio.h>

int main(void) {
    enum month { Jan, Feb, Mar, Apr, May, Jun,
                 Jul, Aug, Sep, Oct, Nov, Dec };

    printf("Jan = %d\n", Jan);  // 0
    printf("Feb = %d\n", Feb);  // 1
    printf("Mar = %d\n", Mar);  // 2
    // ... and so on up to Dec = 11
    return 0;
}
```

> **Note:** `Jan` (an enumeration constant) is an **integer** with value 0.  
> `"Jan"` (a string literal) is a **character array** of 3 characters `{'J','a','n','\0'}`.  
> Do not confuse them!

### Using `typedef` with `enum`

```c
typedef enum { SUN=0, MON, TUE, WED, THU, FRI, SAT } Day;

Day today = WED;   // No need to write "enum Day"

// Works in switch statements:
switch (today) {
    case MON: printf("Start of the week!\n"); break;
    case FRI: printf("Almost weekend!\n");    break;
    default:  printf("Midweek.\n");
}
```

### Anonymous Enum (constants only)

When you only need the constants and not the type name:

```c
enum { RED=0, GREEN, BLUE };   // No tag, just constants
int color = GREEN;             // color == 1
```

### Key Rules

- Enum constants **share the same namespace as ordinary identifiers** — don't reuse them.
- You **cannot directly print** an enum constant as text; you must map it yourself.
- An enum variable internally stores an `int`; you can cast between them.

```c
enum Day d = WED;
int n = (int) d;          // n == 3
enum Day back = (enum Day) 3;   // back == WED
```

---

## 4. Structures (`struct`)

### 4.1 Declaring a Structure

A **structure** groups related variables (called **fields** or **members**) of possibly different types under one name.

> **Mental model:** a struct is like a paper form — it has several labelled boxes, each holding one piece of information.

```
┌──────────────────────────────────────────┐
│  fraction                                │
│  ┌──────────────┐  ┌───────────────────┐ │
│  │  numerator   │  │   denominator     │ │
│  └──────────────┘  └───────────────────┘ │
└──────────────────────────────────────────┘
```

#### Two ways to declare

**Method 1 — Tagged struct (requires `struct` keyword every time you use it):**

```c
struct STUDENT {
    char id[10];
    char name[26];
    int  gradePts;
};                            // semicolon is mandatory!

// To declare a variable:
struct STUDENT aStudent;
```

**Method 2 — `typedef struct` (cleaner, recommended):**

```c
typedef struct {
    char id[10];
    char name[26];
    int  gradePts;
} STUDENT;                    // STUDENT is now the type name

// To declare a variable — much simpler:
STUDENT aStudent;
```

Both styles produce identical variables; `typedef` just saves you writing `struct` everywhere.

### 4.2 Initializing a Structure

Like arrays, structs can be initialized with a brace-enclosed list in **field order**:

```c
typedef struct {
    int   x;
    int   y;
    float t;
    char  u;
} SAMPLE;

SAMPLE sam1 = {2, 5, 3.2f, 'A'};   // all four fields set
SAMPLE sam2 = {7, 3};               // t → 0.0f, u → '\0' (zero-filled)
```

Memory layout of `sam1`:

```
┌─────┬─────┬───────┬───┐
│  2  │  5  │  3.2  │ A │
│  x  │  y  │   t   │ u │
└─────┴─────┴───────┴───┘
```

### 4.3 Accessing Fields — The Dot Operator

Use the **dot operator** (`.`) to access a field when you have a **struct variable** (not a pointer):

```c
STUDENT s;
s.gradePts = 90;
printf("Name: %s\n", s.name);
scanf("%s", s.id);
```

> **Rule:** `variable.field` — left side is a struct value → use `.`

### 4.4 Copying a Structure

You can copy an entire struct with the assignment operator `=`. C copies **all fields** at once:

```c
SAMPLE sam1 = {2, 5, 3.2f, 'A'};
SAMPLE sam2 = {7, 3, 0.0f, 'R'};

sam2 = sam1;   // sam2 now contains {2, 5, 3.2f, 'A'}
```

Memory before and after:

```
Before:                        After sam2 = sam1:
sam2: 7 | 3 | 0.0 | R         sam2: 2 | 5 | 3.2 | A
sam1: 2 | 5 | 3.2 | A         sam1: 2 | 5 | 3.2 | A  (unchanged)
```

### 4.5 Pointers to Structures — The Arrow Operator

When you have a **pointer** to a struct, you use the **arrow operator** (`->`) to reach a field:

```c
SAMPLE  sam1 = {2, 5, 3.2f, 'A'};
SAMPLE* ptr  = &sam1;

// Three equivalent ways to access field x:
int v1 = sam1.x;      // direct access — dot
int v2 = (*ptr).x;    // dereference then dot
int v3 = ptr->x;      // arrow (shorthand for (*ptr).x)
```

> **The golden rule:**
> - Left of accessor is a **struct value** → use **`.`**  
> - Left of accessor is a **struct pointer** → use **`->`**

```
sam1.x   ↔   (*ptr).x   ↔   ptr->x
```

#### Memory Diagram

```
Stack:
┌──────────────────────────┐     ┌─────────────────┐
│ ptr  │ address of sam1 ──┼────►│ sam1: x=2, y=5  │
└──────────────────────────┘     │       t=3.2, u=A │
                                 └─────────────────┘
```

#### Common Mistake

```c
// WRONG — *ptr.x is parsed as *(ptr.x) because . binds tighter than *
int bad = *ptr.x;

// CORRECT — force dereference first
int ok  = (*ptr).x;   // or simply ptr->x
```

### 4.6 Arrays Inside Structures

A field can be an array:

```c
typedef struct {
    char name[26];
    int  midterm[3];
    int  final;
} STUDENT;

STUDENT s;
s.midterm[0] = 85;   // first midterm score
s.midterm[1] = 90;
s.midterm[2] = 78;
printf("First midterm: %d\n", s.midterm[0]);
```

Accessing `s.midterm[1]` means: "go to `s`, then to the `midterm` array field, then to index 1."

### 4.7 Arrays of Structures

You can declare an array where each element is a struct — the most common pattern for managing collections of records:

```c
struct person {
    char   first[32];
    char   last[32];
    int    year;
    double ppg;
};

struct person class[54];   // array of 54 persons

class[0].year = 2006;
class[0].ppg  = 5.2;
strcpy(class[0].first, "Jane");
strcpy(class[0].last,  "Doe");
```

#### Using a Pointer to Traverse

```c
STUDENT stuAry[5] = {
    {"Alice", {85,90,78}, 92},
    {"Bob",   {70,75,80}, 88},
    // ...
};

for (STUDENT* p = stuAry; p < stuAry + 5; p++) {
    printf("%-26s  %d\n", p->name, p->final);
}
```

Note: when `p` is a pointer into an array of structs, use `->` to access fields.

#### Size of an Array of Structs

```c
struct employee {
    int   id;
    char  name[5];
    float salary;
};
struct employee emp[2];

// sizeof a single employee: 4 + 5 + 4 = 13 bytes (before padding)
// sizeof emp[2]:  26 bytes (before padding)
```

### 4.8 Nested Structures

A field can be another struct (declared before the outer one):

```c
struct name {
    char first[32];
    char last[32];
};

struct person {
    int         age;
    float       ppg;
    struct name title;   /* nested — "name" must be defined above */
};

struct person boss;
boss.age = 80;
boss.ppg = 0.1f;
strcpy(boss.title.first, "Dean");
strcpy(boss.title.last,  "Smith");
```

Accessing deeply nested fields uses a chain of dots: `boss.title.first`.

#### Visual

```
stamp
┌────────────────────────────────────────────────┐
│  date                    │  time               │
│ ┌────────┬──────┬──────┐ │ ┌──────┬─────┬───┐ │
│ │ month  │ day  │ year │ │ │ hour │ min │sec│ │
│ └────────┴──────┴──────┘ │ └──────┴─────┴───┘ │
└────────────────────────────────────────────────┘

Access: stamp.date.month   stamp.time.sec
```

### 4.9 Structure Padding

Modern processors read memory in **word-sized** chunks (4 bytes on 32-bit, 8 bytes on 64-bit). To ensure each field starts on an aligned boundary the compiler inserts **padding bytes**:

```c
struct student {
    char a;   // 1 byte — then 1 byte padding inserted
    char b;   // 1 byte — then 2 bytes padding inserted
    int  c;   // 4 bytes — aligned to 4-byte boundary
};
// sizeof(struct student) = 8 bytes (not 6!)
```

Memory layout (32-bit word):

```
Byte: [a] [b] [pad] [pad] [c c c c]
       0    1    2     3    4 5 6 7
```

> **Why does this matter?** When you write `sizeof(myStruct)` you get the padded size, not the sum of individual field sizes. This matters for binary file I/O and network protocols.

### 4.10 Structures and Functions

#### Passing Members (individual fields)

```c
int multiply(int x, int y) { return x * y; }

FRACTION fr1 = {2, 6}, fr2 = {7, 4}, res;
res.numerator   = multiply(fr1.numerator,   fr2.numerator);
res.denominator = multiply(fr1.denominator, fr2.denominator);
```

#### Passing the Whole Struct **by Value**

The entire struct is **copied** into the function's stack frame. Changes inside the function do **not** affect the original.

```c
FRACTION multFr(FRACTION f1, FRACTION f2) {
    FRACTION r;
    r.numerator   = f1.numerator   * f2.numerator;
    r.denominator = f1.denominator * f2.denominator;
    return r;   // the result struct is also copied back
}
```

**Drawback:** for large structs, copying is expensive (CPU time + stack space).

#### Passing by Pointer (recommended for large structs)

```c
void multFr(FRACTION* pF1, FRACTION* pF2, FRACTION* pRes) {
    pRes->numerator   = pF1->numerator   * pF2->numerator;
    pRes->denominator = pF1->denominator * pF2->denominator;
}

// Call:
FRACTION fr1 = {2,6}, fr2 = {7,4}, res;
multFr(&fr1, &fr2, &res);
```

**Advantages of pointer passing:**
- No copy — only the address is sent (4 or 8 bytes regardless of struct size).
- The function can modify the original struct.

**Use `const` pointer to allow read-only access without copying:**

```c
void printFr(const FRACTION* p) {
    printf("%d/%d\n", p->numerator, p->denominator);
    // p->numerator = 0;  ← compile error! const protects the struct
}
```

#### Complete Fraction Program (Pass by Value + Return)

```c
#include <stdio.h>

typedef struct {
    int numerator;
    int denominator;
} FRACTION;

FRACTION getFr(void) {
    FRACTION fr;
    printf("Enter fraction (x/y): ");
    scanf("%d/%d", &fr.numerator, &fr.denominator);
    return fr;
}

FRACTION multFr(FRACTION f1, FRACTION f2) {
    FRACTION r;
    r.numerator   = f1.numerator   * f2.numerator;
    r.denominator = f1.denominator * f2.denominator;
    return r;
}

void printFr(FRACTION f1, FRACTION f2, FRACTION res) {
    printf("\n%d/%d * %d/%d = %d/%d\n",
        f1.numerator,  f1.denominator,
        f2.numerator,  f2.denominator,
        res.numerator, res.denominator);
}

int main(void) {
    FRACTION fr1 = getFr();
    FRACTION fr2 = getFr();
    FRACTION res = multFr(fr1, fr2);
    printFr(fr1, fr2, res);
    return 0;
}
```

Sample run:
```
Enter fraction (x/y): 2/6
Enter fraction (x/y): 7/4

2/6 * 7/4 = 14/24
```

---

## 5. Union Types (`union`)

### Concept

A **union** looks like a struct but all fields **share the same memory location**. The union's size equals the size of its **largest member**.

> **Mental model:** a union is like a single parking space that can hold a car, a motorbike, or a truck — but only one at a time.

### Declaration

```c
union Data {
    int    i;
    float  f;
    char   c;
};
```

Memory layout:

```
┌──────────────────────────┐
│  i (4 bytes)             │ ← also f (4 bytes) ← also c (1 byte)
└──────────────────────────┘
sizeof(union Data) == 4
```

### Usage

```c
union Data d;

d.i = 42;
printf("i = %d\n", d.i);   // 42

d.f = 3.14f;
printf("f = %f\n", d.f);   // 3.14 — valid
printf("i = %d\n", d.i);   // UNDEFINED — f overwrote i's bytes!
```

> **Critical rule:** only the **last assigned member** is valid to read. Reading a different member gives meaningless bytes.

### `typedef` with Union

```c
typedef union {
    int   integer;
    float real;
    char  character;
} NUMBER;

NUMBER n;
n.real = 2.718f;
printf("%.3f\n", n.real);
```

### When to Use Unions

- **Memory savings** when you know only one field is active at a time.
- **Type punning** (interpreting the same bytes as different types) — common in embedded systems and network protocols.
- Often combined with an **enum tag** to track which field is currently active:

```c
typedef enum { INT_TYPE, FLOAT_TYPE, CHAR_TYPE } NumType;

typedef struct {
    NumType type;   // tag — tells us which union field is valid
    union {
        int   i;
        float f;
        char  c;
    } value;
} TaggedNumber;

TaggedNumber n;
n.type    = FLOAT_TYPE;
n.value.f = 3.14f;

// Safe reading:
if (n.type == FLOAT_TYPE)
    printf("Float: %f\n", n.value.f);
```

### Struct vs Union — Key Differences

| Feature | `struct` | `union` |
|---|---|---|
| Memory | Each field has its **own** space | All fields **share** one space |
| Size | Sum of all field sizes (+ padding) | Size of the **largest** field |
| Fields active simultaneously | **All** fields valid | Only the **last written** field valid |
| Use case | Group related data | Store one of several types |

---

## 6. Putting It All Together — Complete Programs

### Clock Simulation with Struct + Pointers

```c
#include <stdio.h>

typedef struct {
    int hr;
    int min;
    int sec;
} CLOCK;

void increment(CLOCK* clock) {
    (clock->sec)++;
    if (clock->sec == 60) {
        clock->sec = 0;
        (clock->min)++;
        if (clock->min == 60) {
            clock->min = 0;
            (clock->hr)++;
            if (clock->hr == 24)
                clock->hr = 0;
        }
    }
}

void show(CLOCK* clock) {
    printf("%02d:%02d:%02d\n", clock->hr, clock->min, clock->sec);
}

int main(void) {
    CLOCK clock = {14, 38, 56};   // initialize to 14:38:56

    for (int i = 0; i < 6; i++) {
        increment(&clock);
        show(&clock);
    }
    return 0;
}
```

Output:
```
14:38:57
14:38:58
14:38:59
14:39:00
14:39:01
14:39:02
```

### Sorting an Array of Structs by Salary

```c
#include <stdio.h>
#include <string.h>

struct Employee {
    int   id;
    char  name[20];
    float salary;
};

void sortBySalary(struct Employee e[], int n) {
    struct Employee temp;
    for (int i = 0; i < n - 1; i++)
        for (int j = i + 1; j < n; j++)
            if (e[i].salary > e[j].salary) {
                temp = e[i]; e[i] = e[j]; e[j] = temp;
            }
}

int main(void) {
    struct Employee emp[5] = {
        {101, "AAAA", 50000},
        {102, "XXXX", 30000},
        {103, "NNNN", 60000},
        {104, "BBBB", 90000},
        {105, "DDDD", 25000}
    };

    sortBySalary(emp, 5);

    for (int i = 0; i < 5; i++)
        printf("%d\t%s\t%.2f\n", emp[i].id, emp[i].name, emp[i].salary);
    return 0;
}
```

Output:
```
105   DDDD   25000.00
102   XXXX   30000.00
101   AAAA   50000.00
103   NNNN   60000.00
104   BBBB   90000.00
```

---

## 7. Guided Exercises (with Solutions)

These exercises come from **Exercise Session 6**. Read each problem carefully, attempt a solution, then check the answer below.

---

### Exercise 1 — The `.` vs `->` Rule

**Given code:**

```c
typedef struct {
    int age;
    int student_id;
} Student;

Student  s1;
Student* p = &s1;
```

**Part a) Fill in `.` or `->`:**

| Expression | Operator | Reason |
|---|---|---|
| `s1___age = 20;` | `.` | `s1` is a **struct value** |
| `p___age = 20;` | `->` | `p` is a **pointer** to struct |
| `(*p)___age = 20;` | `.` | `(*p)` dereferences `p` → result is a **struct value** |

**Part b) General rule:**

> If the expression to the left of the field accessor is of type **`struct T`** (a value), use a **dot**.  
> If it is of type **`struct T*`** (a pointer), use an **arrow**.

**Part c) Equivalence:**

```c
p->age = 25;     // syntactic sugar for:
(*p).age = 25;   // dereference pointer, then access field
```

Yes, they are **exactly equivalent**. `->` was invented precisely to avoid writing `(*p).` every time.

**Part d) Memory diagram:**

```
Stack:
┌───────────────────────────────────────────────┐
│  s1  │ age=??  │ student_id=??  │   ← Student │
│       @ address 0x100 (example)               │
│                                               │
│  p   │   0x100   │              │   ← Student*│
└───────────────────────────────────────────────┘
        │
        └──── points to s1
```

---

### Exercise 2 — Passing Structs to Functions

```c
void set_age_val(Student s, int new_age) {
    s.age = new_age;   // modifies the LOCAL COPY
}

void set_age_ptr(Student* s, int new_age) {
    s->age = new_age;  // modifies the ORIGINAL via pointer
}
```

**Part a):**

```c
Student s1;
s1.age = 0;

set_age_val(s1, 18);
printf("%d\n", s1.age);   // prints 0 — original unchanged!

set_age_ptr(&s1, 18);
printf("%d\n", s1.age);   // prints 18 — original modified!
```

**Part b):** With 50 fields, passing by value copies all 50 fields onto the stack at every call. This wastes stack space and CPU time. Passing a pointer (8 bytes on 64-bit) is always cheap regardless of struct size.

**Part c) Rewrite set_age_ptr:**

```c
void set_age_ptr(Student* s, int new_age) {
    s->age        = new_age;
    s->student_id = 0;
}
```

**Part d)** Use `const Student*` when the function only **reads** the struct and should never modify it. The compiler enforces this, preventing accidental mutations and documenting intent:

```c
void print_student(const Student* s) {
    printf("Age: %d, ID: %d\n", s->age, s->student_id);
    // s->age = 0;  ← compiler error — const protects!
}
```

---

### Exercise 3 — Returning a Struct from a Function

```c
typedef struct { float x; float y; } Point;
```

**Part a) Heap-allocating a Point:**

```c
#include <stdlib.h>
#include <math.h>

Point* new_point(float x, float y) {
    Point* p = (Point*) malloc(sizeof(Point));
    if (p == NULL) { /* handle allocation failure */ return NULL; }
    p->x = x;
    p->y = y;
    return p;
}
```

**Part b) Euclidean distance:**

```c
float distance(Point* a, Point* b) {
    float dx = b->x - a->x;
    float dy = b->y - a->y;
    return sqrtf(dx*dx + dy*dy);
}
```

**Part c) main:**

```c
int main(void) {
    Point* p1 = new_point(1.0f, 2.0f);
    Point* p2 = new_point(4.0f, 6.0f);
    printf("Distance: %.4f\n", distance(p1, p2));   // 5.0000
    free(p1);
    free(p2);
    return 0;
}
```

Distance = √((4−1)² + (6−2)²) = √(9+16) = √25 = **5.0**

**Part d)** You could return a local `Point` by value (`return localPoint;`). This is **correct** — C copies the struct into the caller's context before the stack frame is destroyed. It is not a dangling reference because the value is copied. However, returning a heap pointer is necessary when you want the caller to hold it long-term, or when the struct is very large and you want to avoid copying.

---

### Exercise 5 — Dynamic Array of Structs

```c
typedef struct {
    char name[32];
    int  grade;
} Student;
```

**Part a) Allocate n students:**

```c
Student* roster = (Student*) malloc(n * sizeof(Student));
```

`malloc` returns `void*`; the cast to `Student*` tells the compiler the element type.

**Part b)** `roster[2]` desugars to `*(roster + 2)`, which is a **struct value** (not a pointer). Therefore use **`.`**: `roster[2].grade`.

**Part c) Average grade:**

```c
float average_grade(Student* roster, int n) {
    float sum = 0.0f;
    for (int i = 0; i < n; i++)
        sum += roster[i].grade;
    return (n > 0) ? sum / n : 0.0f;
}
```

**Part d) Best student:**

```c
Student* best_student(Student* roster, int n) {
    if (n == 0) return NULL;
    Student* best = &roster[0];
    for (int i = 1; i < n; i++)
        if (roster[i].grade > best->grade)
            best = &roster[i];
    return best;
}
```

**Part e)** **No** — you must **not** `free` the returned pointer separately. It points **into** the roster array, not to independently allocated memory. Freeing it would corrupt the heap. Only `free(roster)` should be called, once.

---

### Exercise 6 — Dynamic Array of Struct Pointers

**Part a) Two-level allocation:**

```c
Student** roster = (Student**) malloc(n * sizeof(Student*));
for (int i = 0; i < n; i++)
    roster[i] = (Student*) malloc(sizeof(Student));
```

**Part b)**  `roster[0]` is a `Student*` (a pointer), so use **`->`**:

```c
roster[0]->grade = 15;
```

**Part c) Contrast:**

- `Student* roster` — one contiguous block; elements accessed by index, fast cache performance.
- `Student** roster` — each element is a separate allocation, scattered in memory; but you can **reorder** elements by swapping pointers (O(1) per swap) without moving any struct data. Ideal for sorting a large dataset when copying structs would be expensive.

**Part d) Free nested:**

```c
for (int i = 0; i < n; i++)
    free(roster[i]);   // free each individual student first
free(roster);          // then free the pointer array
```

Always free **inner** allocations before the **outer** one.

---

### Exercise 7 — Student Grade Book

**Type definitions:**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

typedef struct {
    int day, month, year;
} Date;

typedef struct {
    char  name[32];
    Date  enrolment;
    float gpa;
    bool  is_active;
} Student;

typedef struct {
    Student* students;
    int      n;
    int      capacity;
} GradeBook;
```

**Part d) new_gradebook:**

```c
GradeBook* new_gradebook(int capacity) {
    GradeBook* gb = (GradeBook*) malloc(sizeof(GradeBook));
    gb->students  = (Student*)   malloc(capacity * sizeof(Student));
    gb->n         = 0;
    gb->capacity  = capacity;
    return gb;
}
```

**Part e) add_student:**

```c
void add_student(GradeBook* gb, Student s) {
    gb->students[gb->n] = s;
    gb->n++;
}
```

**Part f) print_gradebook:**

```c
void print_gradebook(GradeBook* gb) {
    for (int i = 0; i < gb->n; i++)
        printf("%-31s  GPA: %.2f\n",
               gb->students[i].name,
               gb->students[i].gpa);
}
```

**Part g) count_active:**

```c
int count_active(GradeBook* gb) {
    int count = 0;
    for (int i = 0; i < gb->n; i++)
        if (gb->students[i].is_active) count++;
    return count;
}
```

**Part h) enrolled_in_month:**

```c
Student* enrolled_in_month(GradeBook* gb, int month, int* result_size) {
    /* First pass — count */
    int count = 0;
    for (int i = 0; i < gb->n; i++)
        if (gb->students[i].enrolment.month == month) count++;

    /* Allocate and fill */
    Student* result = (Student*) malloc(count * sizeof(Student));
    int idx = 0;
    for (int i = 0; i < gb->n; i++)
        if (gb->students[i].enrolment.month == month)
            result[idx++] = gb->students[i];

    *result_size = count;
    return result;
    /* The CALLER must free the returned array */
}
```

**Part i) Free sequence:**

```c
free(gb->students);   // free the internal array first
free(gb);             // then free the GradeBook itself
```

---

## 8. Quick-Reference Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════╗
║              C DERIVED TYPES — QUICK REFERENCE                  ║
╠══════════════════════════════════════════════════════════════════╣
║ TYPEDEF                                                         ║
║   typedef existing_type  NEW_NAME;                              ║
║   typedef int Int32;                                            ║
╠══════════════════════════════════════════════════════════════════╣
║ ENUM                                                            ║
║   enum Tag { A=0, B, C };          // A=0, B=1, C=2            ║
║   typedef enum { A, B, C } MyEnum; // cleaner                  ║
║   MyEnum x = B;                                                 ║
╠══════════════════════════════════════════════════════════════════╣
║ STRUCT                                                          ║
║   typedef struct {                                              ║
║       int   field1;                                             ║
║       float field2;                                             ║
║   } MyStruct;                                                   ║
║                                                                 ║
║   MyStruct  s  = {1, 2.0f};   // by value                      ║
║   MyStruct* p  = &s;          // pointer                        ║
║                                                                 ║
║   s.field1     // value access → dot                           ║
║   p->field1    // pointer access → arrow                        ║
║   (*p).field1  // same as above                                 ║
╠══════════════════════════════════════════════════════════════════╣
║ UNION                                                           ║
║   typedef union {                                               ║
║       int   i;                                                  ║
║       float f;                                                  ║
║   } Number;                                                     ║
║                                                                 ║
║   Number n;  n.f = 3.14f;   // only n.f is valid now           ║
╠══════════════════════════════════════════════════════════════════╣
║ KEY RULES                                                       ║
║   • Use . when left side is a struct VALUE                      ║
║   • Use -> when left side is a struct POINTER                   ║
║   • p->x   ≡   (*p).x   (always)                               ║
║   • Pass struct by pointer to avoid copying large structs       ║
║   • Use const T* to signal read-only access                     ║
║   • In a union only the last-written field is valid             ║
║   • struct size = sum of fields + padding (compiler-added)      ║
║   • Free inner allocations before outer ones                    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

*End of Tutorial — Enumerated, Structure, and Union Types in C*  
*Systems, Algorithms and Programming – 2 | French-Azerbaijani University*
