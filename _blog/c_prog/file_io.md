---
title: "File I/O"
layout: single
author_profile: true
mathjax: true
---

# File Input/Output in C
### A Complete Zero-to-Mastery Tutorial

---

## Table of Contents

1. [What Is a File?](#1-what-is-a-file)
2. [Streams — The Bridge Between Your Program and the World](#2-streams)
3. [Standard Library I/O Functions Overview](#3-standard-library-io-functions)
4. [Working with Files: Open and Close](#4-working-with-files-open-and-close)
   - [Declaring a File Pointer](#declaring-a-file-pointer)
   - [Opening a File: `fopen()`](#opening-a-file-fopen)
   - [File Opening Modes](#file-opening-modes)
   - [Closing a File: `fclose()`](#closing-a-file-fclose)
   - [Handling Open/Close Errors](#handling-openclose-errors)
5. [Formatting Input/Output Functions](#5-formatting-inputoutput-functions)
   - [Terminal vs. General I/O](#terminal-vs-general-io)
   - [Writing to a File: `fprintf()`](#writing-to-a-file-fprintf)
   - [Reading from a File: `fscanf()`](#reading-from-a-file-fscanf)
   - [Conversion Specifications Cheat Sheet](#conversion-specifications-cheat-sheet)
6. [Common Pitfalls and Errors](#6-common-pitfalls-and-errors)
7. [Complete Working Programs](#7-complete-working-programs)
   - [Program 1 — Write an Integer to a Text File](#program-1--write-an-integer-to-a-text-file)
   - [Program 2 — Read an Integer from a Text File](#program-2--read-an-integer-from-a-text-file)
   - [Program 3 — Read and Print All Integers in a File](#program-3--read-and-print-all-integers-in-a-file)
   - [Program 4 — Copy a Text File](#program-4--copy-a-text-file)
   - [Program 5 — Append Data to a File](#program-5--append-data-to-a-file)
   - [Program 6 — Student Grades System (Full Project)](#program-6--student-grades-system-full-project)
8. [Binary Files with `fwrite()` and `fread()`](#8-binary-files)
9. [Key Rules to Remember](#9-key-rules-to-remember)
10. [Practice Exercises](#10-practice-exercises)

---

## 1. What Is a File?

So far in your C programs, all data lives in variables — which means it disappears the moment the program ends. If you want data to **survive between runs**, you need a **file**.

> **Definition:** A file is an external collection of related data treated as a unit. Its primary purpose is to keep a permanent record of data.

Think of a file as a notebook stored on disk. Your program can open it, read from it, write to it, and close it — and everything written remains there even after the computer shuts down.

**Key distinction:**
- **Primary memory (RAM):** fast, temporary — lost on shutdown.
- **Secondary storage (disk/SSD):** slower, but permanent — files live here.

---

## 2. Streams

In C, you do not read from or write to a file directly. Instead, you communicate through a **stream** — an abstract flow of data connecting your program to a data source or destination.

```
Data Source          Program
┌────────────┐       ┌──────────────────────────┐
│ Keyboard / │──────▶│   Input Text Stream  ──▶ Data │
│   File     │       └──────────────────────────┘
│            │
│ Monitor /  │◀──────┌──────────────────────────┐
│   File     │       │  Output Text Stream ◀── Data │
└────────────┘       └──────────────────────────┘
```

### Text vs. Binary Streams

| Stream Type | What it stores | Human-readable? |
|-------------|---------------|-----------------|
| **Text**    | Characters, newlines, formatted data | ✅ Yes |
| **Binary**  | Raw bytes, exact memory representation | ❌ No |

This tutorial focuses primarily on **text streams**, which are the most common for beginners.

### System-Created (Standard) Streams

When your C program starts, three streams are **automatically opened** by the operating system — you never need to open or close them yourself:

| Name     | Meaning          | Connected to    |
|----------|------------------|-----------------|
| `stdin`  | Standard Input   | Keyboard        |
| `stdout` | Standard Output  | Monitor/Terminal|
| `stderr` | Standard Error   | Monitor/Terminal|

> **Important Note:** Standard stream names are already declared in `<stdio.h>`. You must not redeclare them. You also never need to open or close them — the OS handles this automatically.

---

## 3. Standard Library I/O Functions

All file I/O in C uses functions declared in `<stdio.h>`. These functions are organised into **eight categories**:

| Category                | Purpose |
|-------------------------|---------|
| **File Open/Close**     | `fopen()`, `fclose()` — connect/disconnect your program from a file |
| **Formatted I/O**       | `fprintf()`, `fscanf()` — read/write with format strings |
| **Character I/O**       | `fgetc()`, `fputc()` — read/write one character at a time |
| **Line I/O**            | `fgets()`, `fputs()` — read/write entire lines |
| **Block I/O**           | `fread()`, `fwrite()` — read/write raw binary blocks |
| **File Positioning**    | `fseek()`, `ftell()`, `rewind()` — move the file marker |
| **System File Ops**     | `remove()`, `rename()` — delete or rename files |
| **File Status**         | `feof()`, `ferror()` — check stream state |

In this tutorial we will master the first two categories in depth, with an introduction to block I/O for binary files.

---

## 4. Working with Files: Open and Close

Every file operation follows the same three-step pattern:

```
1. OPEN  the file   →  fopen()
2. USE   the file   →  fscanf() / fprintf() / etc.
3. CLOSE the file   →  fclose()
```

Never skip step 3. Failing to close a file can cause data loss or corruption.

---

### Declaring a File Pointer

Before you can open a file, you need a **file pointer** — a variable of type `FILE *`. This pointer points to a structure that C uses internally to manage the stream.

```c
FILE *fptr;   // declares a file pointer named fptr
```

You can name it anything you like (`spData`, `spIn`, `spOut`, `fp`, etc.). The convention `sp` stands for *stream pointer*, which is a common naming style.

---

### Opening a File: `fopen()`

```c
FILE *fopen(const char *filename, const char *mode);
```

- `filename` — the name of the file on disk (can include a path).
- `mode` — a string telling C how you want to use the file.
- **Returns** a `FILE *` on success, or `NULL` on failure.

**Example:**

```c
FILE *spData;
spData = fopen("MYDATA.DAT", "w");
```

What happens here:

```
"MYDATA.DAT"         →  external file on disk
"w"                  →  open for writing
spData               →  internal file pointer your code uses
```

The file pointer `spData` is the handle your program uses from this point on. The actual physical file `MYDATA.DAT` lives on disk.

---

### File Opening Modes

There are three fundamental text file modes:

| Mode | Full Name | File exists behaviour | File does NOT exist |
|------|-----------|----------------------|---------------------|
| `"r"` | Read | Opens it; marker at **beginning** | Returns `NULL` ❌ |
| `"w"` | Write | **Erases** existing contents; marker at beginning | **Creates** the file ✅ |
| `"a"` | Append | Opens it; marker at **end** | **Creates** the file ✅ |

Visualised:

```
Mode "r" — Read:              Mode "w" — Write:          Mode "a" — Append:
┌──────────────────┐[EOF]     [EOF]┌───────────────┐     ┌──────────────────┐[EOF]
│  existing data   │              │  (empty)       │     │  existing data   │
└──────────────────┘              └───────────────┘     └──────────────────┘
 ▲ marker starts here               ▲ marker starts here                    ▲ marker starts here
```

> **Critical:** Using `"w"` on an existing file **destroys** all its previous content. Use `"a"` when you want to add to an existing file.

---

### Closing a File: `fclose()`

```c
int fclose(FILE *stream);
```

- Flushes any buffered data to disk.
- Releases the file from your program.
- Returns `0` on success, `EOF` on failure.

```c
fclose(spData);
```

---

### Handling Open/Close Errors

Always check whether `fopen()` succeeded. If it returns `NULL`, the file could not be opened — perhaps it does not exist (for read mode), or the disk is full (for write mode).

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    FILE *spTemps;

    // Try to open the file for reading
    if ((spTemps = fopen("TEMPS.DAT", "r")) == NULL)
    {
        printf("\aERROR opening TEMPS.DAT\n");
        exit(100);   // exit with an error code
    }

    // ... use the file ...

    // Try to close the file
    if (fclose(spTemps) == EOF)
    {
        printf("\aERROR closing TEMPS.DAT\n");
        exit(102);
    }

    return 0;
}
```

**Why `exit()`?** When a critical file cannot be opened, there is no point continuing. `exit()` from `<stdlib.h>` terminates the program immediately with the given status code.

---

## 5. Formatting Input/Output Functions

You already know `scanf` and `printf` from console programming. For files, C provides their more powerful siblings:

### Terminal vs. General I/O

| Function | Reads/Writes from/to | Syntax |
|----------|----------------------|--------|
| `scanf`  | `stdin` (keyboard) only | `scanf("format", ...);` |
| `printf` | `stdout` (screen) only  | `printf("format", ...);` |
| `fscanf` | **any** stream (file or standard) | `fscanf(stream_ptr, "format", ...);` |
| `fprintf`| **any** stream (file or standard) | `fprintf(stream_ptr, "format", ...);` |

The only syntactic difference is that `fscanf`/`fprintf` take a **stream pointer as their first argument**.

> **Note:** `scanf` is equivalent to `fscanf(stdin, ...)` and `printf` is equivalent to `fprintf(stdout, ...)`.

---

### Writing to a File: `fprintf()`

```c
fprintf(file_pointer, "format string", variable1, variable2, ...);
```

**Example — write an integer and a float:**

```c
FILE *spOut = fopen("output.txt", "w");

int age = 21;
float gpa = 3.75;

fprintf(spOut, "Age: %d, GPA: %.2f\n", age, gpa);

fclose(spOut);
```

The file `output.txt` will contain:
```
Age: 21, GPA: 3.75
```

`fprintf` converts internal data to character strings and writes them to the file. It returns the number of characters written, or `EOF` on error.

---

### Reading from a File: `fscanf()`

```c
fscanf(file_pointer, "format string", &variable1, &variable2, ...);
```

**Example — read integers until end of file:**

```c
FILE *spIn = fopen("data.txt", "r");
int num;

while (fscanf(spIn, "%d", &num) == 1)
{
    printf("Read: %d\n", num);
}

fclose(spIn);
```

`fscanf` reads characters from the file and converts them to the requested data types. It returns:
- The **number of successful conversions** (e.g., `1` if one value was read).
- `EOF` if the end of the file is reached before any conversion.

The `== 1` check in the loop above ensures we keep reading as long as there is a valid integer to read.

---

### Conversion Specifications Cheat Sheet

Both `fscanf` and `fprintf` use **format strings** with **conversion specifications** that begin with `%`.

**For `fprintf` (output):**

```
% [flag] [min_width] [.precision] [size] conversion_code
```

| Flag | Meaning |
|------|---------|
| `-`  | Left-justify within field |
| `+`  | Always print sign (`+` or `-`) |
| ` `  | Print a space before positive numbers |
| `0`  | Pad with zeros instead of spaces |

**For `fscanf` (input):**

```
% [*] [max_width] [size] conversion_code
```

| Flag | Meaning |
|------|---------|
| `*`  | Suppress — read but do NOT store the value |

**Common conversion codes (both functions):**

| Code | Data Type | Example |
|------|-----------|---------|
| `%d` | `int` (decimal) | `42` |
| `%u` | `unsigned int` | `42` |
| `%f` | `float` / `double` | `3.14` |
| `%e` | Scientific notation | `3.14e+00` |
| `%c` | Single character | `'A'` |
| `%s` | String (char array) | `"hello"` |
| `%o` | Octal integer | `052` |
| `%x` | Hexadecimal integer | `0x2A` |

**Size modifiers:**

| Modifier | Meaning |
|----------|---------|
| `h`  | `short int` |
| `l`  | `long int` or `double` (with `%f`) |
| `L`  | `long double` |
| `hh` | `char` |
| `ll` | `long long` |

**Practical examples:**

```c
// Output: integer padded to 4 digits with leading zeros
fprintf(fp, "%04d", 42);        // → "0042"

// Output: float with 2 decimal places
fprintf(fp, "%.2f", 3.14159);   // → "3.14"

// Output: left-justified string in 10-char field
fprintf(fp, "%-10s", "hello");  // → "hello     "

// Input: read a long integer
fscanf(fp, "%ld", &bigNum);

// Input: read a double
fscanf(fp, "%lf", &price);
```

> **Critical rule:** The number, order, and type of conversion specifications must exactly match the variables in the argument list. A mismatch produces unpredictable results.

---

### Whitespace Behaviour

- In an **input** format string, any whitespace character (space, `\t`, `\n`) causes `fscanf` to skip all leading whitespace in the input — including newlines. This is usually what you want.
- In an **output** format string, whitespace is copied literally to the output stream.

---

## 6. Common Pitfalls and Errors

### Pitfall 1 — Missing `&` (address operator) in `fscanf`

```c
// WRONG — 'a' is passed as a value, not an address
fscanf(fp, "%d", a);

// CORRECT — pass the address of 'a'
fscanf(fp, "%d", &a);
```

Missing `&` is one of the most common C bugs. `fscanf` needs to **write** into the variable, so it must receive a pointer to it.

---

### Pitfall 2 — Type mismatch in format string

```c
int a;
// WRONG — %f expects float/double, but a is int
fscanf(fp, "%f", &a);

// CORRECT
fscanf(fp, "%d", &a);
```

If the format code does not match the variable type, `fscanf` reads the wrong number of bytes and corrupts memory.

---

### Pitfall 3 — Two integers run together without a separator

If your file contains `23r` and your format is `"%d%d"`, the first `%d` will read `23`, then the second `%d` will fail because `r` is not a digit. The result stored in the second variable is undefined.

Always design your file format so numbers are separated by whitespace or delimiters.

---

### Pitfall 4 — Not checking `fopen()` return value

```c
// DANGEROUS — if fopen returns NULL, the next call crashes
FILE *fp = fopen("data.txt", "r");
fscanf(fp, "%d", &num);   // undefined behaviour if fp == NULL!
```

Always check for `NULL`:

```c
if ((fp = fopen("data.txt", "r")) == NULL) {
    printf("Cannot open file!\n");
    exit(1);
}
```

---

### Pitfall 5 — Forgetting to close the file

Data written to a file is often buffered in memory. If you skip `fclose()`, that buffered data may never be written to disk.

---

## 7. Complete Working Programs

---

### Program 1 — Write an Integer to a Text File

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int num;
    FILE *fptr;

    // Open file for writing (creates it if it doesn't exist)
    fptr = fopen("file.txt", "w");
    if (fptr == NULL)
    {
        printf("Error opening file!\n");
        exit(1);
    }

    printf("Enter a number: ");
    scanf("%d", &num);

    // Write the integer to the file
    fprintf(fptr, "%d", num);

    fclose(fptr);
    printf("Number written to file.txt\n");
    return 0;
}
```

**What this does:** Creates (or overwrites) `file.txt`, reads one integer from the keyboard, writes it to the file, then closes it.

---

### Program 2 — Read an Integer from a Text File

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int num;
    FILE *fptr;

    // Open file for reading
    if ((fptr = fopen("file.txt", "r")) == NULL)
    {
        printf("Error opening file!\n");
        exit(1);
    }

    // Read one integer from the file
    fscanf(fptr, "%d", &num);
    printf("Value read from file: %d\n", num);

    fclose(fptr);
    return 0;
}
```

**What this does:** Opens the file written by Program 1, reads the integer back, and prints it.

---

### Program 3 — Read and Print All Integers in a File

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    FILE *ptr;
    int numIn;

    ptr = fopen("P07_03.txt", "r");
    if (!ptr)
    {
        printf("File can't be opened\n");
        return 1;
    }

    // Loop: keep reading integers until fscanf fails to read one
    while (fscanf(ptr, "%d", &numIn) == 1)
    {
        printf("%d\n", numIn);
    }

    fclose(ptr);
    return 0;
}
```

**Key concept — the loop condition:**

```c
while (fscanf(ptr, "%d", &numIn) == 1)
```

`fscanf` returns `1` each time it successfully reads one integer. When the file ends, it returns `EOF` (which is `-1`), and the loop exits cleanly.

**Sample file `P07_03.txt`:**
```
102312  13  15
202313  15  18
302310  12  14
```

**Output:**
```
102312
13
15
202313
15
18
302310
12
14
```

---

### Program 4 — Copy a Text File

This program reads every integer from an input file and writes each one to an output file, one per line.

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    FILE *spIn;
    FILE *spOut;
    int numIn;
    int closeResult;

    printf("Running file copy\n");

    // Open input file
    spIn = fopen("P07_03.txt", "r");
    if (!spIn)
    {
        printf("Cannot open input file\n");
        return 1;
    }

    // Open output file
    spOut = fopen("P07-04.txt", "w");
    if (!spOut)
    {
        printf("Cannot open output file\n");
        fclose(spIn);
        return 1;
    }

    // Copy all integers from input to output
    while (fscanf(spIn, "%d", &numIn) == 1)
    {
        fprintf(spOut, "%d\n", numIn);
    }

    // Close output file and check for error
    closeResult = fclose(spOut);
    if (closeResult == EOF)
    {
        printf("Could not close output file\n");
    }

    fclose(spIn);
    printf("File copy complete\n");
    return 0;
}
```

**Input file (`P07_03.txt`):**
```
102312  13  15
202313  15  18
302310  12  14
```

**Output file (`P07-04.txt`):**
```
102312
13
15
202313
15
18
302310
12
14
```

Notice how the output has each number on its own line (`%d\n`) even though the input had multiple numbers per line. This is because `fscanf` with `%d` skips whitespace (including newlines) between numbers.

---

### Program 5 — Append Data to a File

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    FILE *spAppend;
    int numIn;
    int closeResult;

    printf("This program appends data to a file\n");

    // Open in APPEND mode — adds to end, creates if needed
    spAppend = fopen("P07_05.txt", "a");
    if (!spAppend)
    {
        printf("Could not open file\n");
        return 1;
    }

    printf("Please enter first number (Ctrl+Z / Ctrl+D to stop): ");
    while (scanf("%d", &numIn) != EOF)
    {
        fprintf(spAppend, "%d\n", numIn);
        printf("Enter next number or EOF: ");
    }

    closeResult = fclose(spAppend);
    if (closeResult == EOF)
        printf("Could not close file\n");

    printf("Append complete\n");
    return 0;
}
```

**Sample run:**
```
This program appends data to a file
Please enter first number: 333
Enter next number or EOF: 254
Enter next number or EOF: 123
Enter next number or EOF: ^Z
Append complete
```

After each run, the new numbers are **added to the end** of `P07_05.txt` without erasing previous data. This is the power of append mode.

---

### Program 6 — Student Grades System (Full Project)

This is the most complete example. It demonstrates a real-world file processing program that:
1. Reads student data from an input file.
2. Calculates each student's average and letter grade.
3. Writes the results to an output file.

The program is split into three functions for clean design:

```c
#include <stdio.h>
#include <stdlib.h>

/* ---- Function Declarations ---- */
int  getStu     (FILE *spStu, int *stuID, int *exam1, int *exam2, int *final);
void calcGrade  (int exam1, int exam2, int final, int *avrg, char *grade);
int  writeStu   (FILE *spGrades, int stuID, int avrg, char grade);

/* ====================== MAIN ====================== */
int main(void)
{
    FILE *spStu;      // input  file pointer
    FILE *spGrades;   // output file pointer

    int  stuID, exam1, exam2, final, avrg;
    char grade;

    printf("Begin student grading\n");

    // Open input file
    if (!(spStu = fopen("P07-06.DAT", "r")))
    {
        printf("\aError opening student file\n");
        return 100;
    }

    // Open output file
    if (!(spGrades = fopen("P07-06Gr.DAT", "w")))
    {
        printf("\aError opening grades file\n");
        return 102;
    }

    // Process each student
    while (getStu(spStu, &stuID, &exam1, &exam2, &final))
    {
        calcGrade(exam1, exam2, final, &avrg, &grade);
        writeStu(spGrades, stuID, avrg, grade);
    }

    fclose(spStu);
    fclose(spGrades);

    printf("End student grading\n");
    return 0;
}

/* ====================== getStu ====================== 
   Reads one student's data from the input file.
   Returns 1 if data was read, 0 at EOF or on error.
   ===================================================== */
int getStu(FILE *spStu, int *stuID, int *exam1, int *exam2, int *final)
{
    int ioResult;

    ioResult = fscanf(spStu, "%d%d%d%d", stuID, exam1, exam2, final);

    if (ioResult == EOF)
        return 0;
    else if (ioResult != 4)
    {
        printf("\aError reading data\n");
        return 0;
    }
    else
        return 1;
}

/* ====================== calcGrade ===================
   Computes average and assigns a letter grade.
   Uses absolute grading scale: A≥90, B≥80, C≥70,
   D≥60, F otherwise.
   ===================================================== */
void calcGrade(int exam1, int exam2, int final,
               int *avrg, char *grade)
{
    *avrg = (exam1 + exam2 + final) / 3;

    if      (*avrg >= 90) *grade = 'A';
    else if (*avrg >= 80) *grade = 'B';
    else if (*avrg >= 70) *grade = 'C';
    else if (*avrg >= 60) *grade = 'D';
    else                  *grade = 'F';
}

/* ====================== writeStu ====================
   Writes one student's grade record to the output file.
   Format: stuID (4-digit) average grade
   ===================================================== */
int writeStu(FILE *spGrades, int stuID, int avrg, char grade)
{
    fprintf(spGrades, "%04d %d %c\n", stuID, avrg, grade);
    return 0;
}
```

**Input file (`P07-06.DAT`):**
```
0090 90 90 90
0089 88 90 89
0081 80 82 81
0079 79 79 79
0070 70 70 70
0069 69 69 69
0060 60 60 60
0059 59 59 59
```

**Output file (`P07-06Gr.DAT`):**
```
0090 90 A
0089 89 B
0081 81 B
0079 79 C
0070 70 C
0069 69 D
0060 60 D
0059 59 F
```

**Design notes:**
- `getStu()` returns `1` (true) on success and `0` (false) on EOF or error, which makes it perfect to use directly as a `while` condition.
- The `%04d` format prints the student ID padded to 4 digits with leading zeros.
- By separating reading, calculating, and writing into distinct functions, the code is clean, testable, and reusable.

---

## 8. Binary Files

Binary files store data as raw bytes (the same representation as in memory), rather than as human-readable text. They are more compact and faster to read/write, but cannot be opened in a text editor.

### `fwrite()` — Writing Binary Data

```c
size_t fwrite(const void *ptr, size_t size, size_t count, FILE *stream);
```

- `ptr` — pointer to the data to write.
- `size` — size (in bytes) of each element.
- `count` — number of elements to write.
- Returns the number of elements actually written.

### Example — Writing a Struct to a Binary File

```c
#include <stdio.h>
#include <stdlib.h>

struct threeNum
{
    int n1, n2, n3;
};

int main(void)
{
    int n;
    struct threeNum num;
    FILE *fptr;

    // Open binary file for writing
    if ((fptr = fopen("program.bin", "wb")) == NULL)
    {
        printf("Error opening file!\n");
        exit(1);
    }

    for (n = 1; n < 5; n++)
    {
        num.n1 = n;
        num.n2 = 5 * n;
        num.n3 = 5 * n + 1;

        // Write one struct to the file
        fwrite(&num, sizeof(struct threeNum), 1, fptr);
    }

    fclose(fptr);
    printf("Binary file written.\n");
    return 0;
}
```

The binary file `program.bin` will contain the raw bytes of 4 structs. To read it back, use `fread()` with the same `struct threeNum` type.

**Mode string for binary files:** Use `"wb"` (write binary), `"rb"` (read binary), or `"ab"` (append binary).

---

## 9. Key Rules to Remember

| Rule | Why It Matters |
|------|----------------|
| Always check `fopen()` for `NULL` | Prevents crashes from dereferencing a null pointer |
| Always call `fclose()` | Ensures buffered data is flushed to disk |
| Use `"r"` only for existing files | `"r"` returns `NULL` if the file doesn't exist |
| Use `"w"` carefully | It **erases** any existing file with the same name |
| Use `"a"` to add without erasing | Appends to the end; creates the file if needed |
| Match format codes to variable types | Mismatches cause undefined behaviour |
| Use `&` with `fscanf` arguments | `fscanf` must write into variables, so it needs addresses |
| Check `fscanf` return value in loops | Use `== 1` (or `== n` for n variables) to detect EOF cleanly |
| Standard streams need no open/close | `stdin`, `stdout`, `stderr` are managed by the OS |
| Standard stream names cannot be redeclared | They are already defined in `<stdio.h>` |

---

## 10. Practice Exercises

Work through these exercises in order. Each one builds on the previous.

**Exercise 1 — Basic write/read**  
Write a program that asks the user for their name (a string) and age (an integer), saves them to a file called `person.txt`, then reads the file back and prints the values.

**Exercise 2 — Multi-record file**  
Write a program that writes 5 student records (ID and score) to a file, one per line. Then write a second program that reads the file and prints only students who scored above 70.

**Exercise 3 — Statistics from a file**  
Write a program that reads a file of floating-point numbers (one per line) and computes and prints the minimum, maximum, sum, and average.

**Exercise 4 — File copy with transformation**  
Write a program that reads a text file of integers, doubles each value, and writes the results to a new file. Both file names should be accepted as command-line arguments.

**Exercise 5 — Append and count**  
Write a program that appends a number to `log.txt` each time it runs, then prints how many numbers are in the file so far.

**Exercise 6 — Extend the Student Grades project**  
Modify Program 6 to also print a summary line at the end of the output file showing: total students, number of each grade (A/B/C/D/F), and the class average.

---

## Summary

Here is the complete workflow for C file I/O at a glance:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    FILE *fp;                               // 1. Declare file pointer
    int value;

    fp = fopen("myfile.txt", "w");          // 2. Open the file
    if (fp == NULL) {                       // 3. Always check for NULL
        printf("Error!\n");
        exit(1);
    }

    fprintf(fp, "%d\n", 42);               // 4. Use the file
    fprintf(fp, "%d\n", 99);

    fclose(fp);                             // 5. Always close the file

    // Now read it back
    fp = fopen("myfile.txt", "r");
    if (fp == NULL) { printf("Error!\n"); exit(1); }

    while (fscanf(fp, "%d", &value) == 1)  // 6. Loop until EOF
        printf("Read: %d\n", value);

    fclose(fp);                             // 7. Close again
    return 0;
}
```

You now have everything you need to read, write, copy, and append files in C — from the simplest single-value file to a multi-function data processing system. Practice consistently, always handle errors, and never forget to close your files!

---

*Tutorial based on Chapter 7 — File Input/Output, C Programming course materials.*
