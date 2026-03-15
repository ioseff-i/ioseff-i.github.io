---
title: "Introduction to Processes"
layout: single
author_profile: true
mathjax: true
---

Every program you have ever written was, at some point, a **process**. When you double-click an application, open a terminal, or run `./myprogram`, the operating system takes your code from disk and brings it to life in memory. That living, running instance is called a process. Understanding what a process is, how it is born, how it communicates, and how it dies is the foundation of systems programming, parallel computing, and everything that comes after it — threads, OpenMP, MPI, and CUDA.

This post builds that foundation from scratch.

---

## 1. What Is a Process?

A **process** is a running instance of a program. The program itself is just a file sitting on disk — dead bytes. The moment the OS loads it into memory and starts executing it, it becomes a process: alive, active, consuming CPU time and memory.

```
program on disk:   myprogram   (just bytes, not running)
                       ↓
        OS loads it into memory and runs it
                       ↓
        now it is a PROCESS    (alive, running)
```

Every process gets its own private set of resources from the OS:

- **Memory** — its own stack, heap, and code segment
- **PID** — a unique Process ID number
- **File descriptors** — its own stdin, stdout, open files
- **State** — program counter, CPU registers, variables

The key word is **private**. One process cannot touch another process's memory. They are completely isolated from each other — like two separate computers running at the same time.

---

## 2. The Process ID (PID)

Every process has a unique integer called a **PID** (Process ID). The OS assigns it at birth and uses it to track the process.

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("This is the main process with PID: %d\n", getpid());
    return 0;
}
```

Run it twice:

```
This is the main process with PID: 4821
This is the main process with PID: 4822
```

Different PID every time — because each run is a brand new process.

Two useful functions:

| Function | What it returns |
|---|---|
| `getpid()` | My own PID |
| `getppid()` | My parent's PID |

---

## 3. Creating a New Process — `fork()`

The most fundamental operation in Unix process management is `fork()`. It creates a new process by making an **exact copy** of the current one.

### The Photocopier Analogy

Imagine you have a piece of paper with instructions written on it. You put it into a photocopier. Now two identical papers exist. Both have the same instructions. The photocopier writes a secret mark on each:

- On the **original**: the copy's serial number (e.g. 23324)
- On the **copy**: the number zero

That secret mark is the return value of `fork()`.

### What fork() Actually Does

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("Before fork. PID = %d\n", getpid());

    fork();   // the copy happens here

    printf("After fork. PID = %d\n", getpid());
    return 0;
}
```

Output:

```
Before fork. PID = 100
After fork. PID = 100    ← original (parent) prints this
After fork. PID = 101    ← copy (child) prints this
```

"Before fork" prints **once** — only one process existed.  
"After fork" prints **twice** — two processes continue from that line onwards.

```
BEFORE fork():              AFTER fork():

  Process 100                 Process 100  (PARENT)
  ┌──────────┐                ┌──────────┐
  │ running  │   ──────►      │ running  │
  └──────────┘                └──────────┘
                                    +
                              Process 101  (CHILD)
                              ┌──────────┐
                              │ running  │  ← exact copy
                              └──────────┘
```

### The Magic Return Value

`fork()` is the only function in existence that you call **once** but it **returns twice** — once in the parent, once in the child. Each gets a different value:

```
fork() returns:
  → to the PARENT:   the child's PID   (a positive number)
  → to the CHILD:    0
  → on error:        -1
```

After `fork()`, two completely separate variables named `pid` exist — one in each process's memory:

```
PARENT's memory:              CHILD's memory:
┌──────────────┐              ┌──────────────┐
│ pid = 101    │              │ pid = 0      │
└──────────────┘              └──────────────┘
   (child's PID)                 (just a signal)
```

They are separate. Changing one does not affect the other.

### An Important Clarification

`pid == 0` inside the child does **not** mean the child's PID is zero. Zero is just a signal meaning "I am the newly created one." The child's real PID is always a positive number:

```
Inside child process:
  pid      = 0       ← signal: "I am the child"
  getpid() = 101     ← my actual real PID
```

Think of twins born at a hospital. Each baby gets a wristband:
- First baby's wristband says **"002"** (second baby's ID)
- Second baby's wristband says **"000"** (meaning "I am the new one")

But if you ask each baby their name, they each give their own real name.

---

## 4. Parent and Child — Different Behavior from the Same Code

The return value of `fork()` lets parent and child take different paths through the same code:

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("INITIAL PROCESS with PID: %d\n", getpid());

    pid_t pid = fork();

    if (pid > 0) {
        // only PARENT comes here — pid holds child's PID
        printf("I am the PARENT. My PID=%d, child's PID=%d\n",
               getpid(), pid);
    }
    else if (pid == 0) {
        // only CHILD comes here — pid is 0
        printf("I am the CHILD. My PID=%d, parent's PID=%d\n",
               getpid(), getppid());
    }
    else {
        printf("fork() failed!\n");
    }

    return 0;
}
```

Output:

```
INITIAL PROCESS with PID: 100
I am the PARENT. My PID=100, child's PID=101
I am the CHILD.  My PID=101, parent's PID=100
```

The step-by-step trace:

```
ONE process runs this:
  printf("INITIAL PROCESS 100")

fork() → TWO processes now

PARENT (pid = 101):           CHILD (pid = 0):
  pid > 0 → TRUE                pid == 0 → TRUE
  enters parent block           enters child block
  prints "I am the PARENT"      prints "I am the CHILD"
```

---

## 5. Process Isolation — The Biggest Difference from Threads

Because processes have completely separate memory, changes in one are invisible to the other:

```
PROCESS A                     PROCESS B
┌──────────────────┐          ┌──────────────────┐
│  int x = 5       │          │  int x = 99      │
│  heap memory     │          │  heap memory     │
└──────────────────┘          └──────────────────┘
      ↑                              ↑
 completely                    completely
 separate                       separate

If A changes x to 10 → B still sees x = 99
```

This is fundamentally different from threads, which share memory and can see each other's changes immediately. Isolation makes processes **safe** but means they cannot share variables directly.

---

## 6. Zombie Processes

When a child process finishes, it does not disappear immediately. It enters a **zombie** state.

### What Is a Zombie?

A zombie process has:
- Stopped executing — it is dead
- But its entry in the OS process table is still there
- It is waiting for the parent to collect its exit status

```
Child finishes running
          ↓
Child becomes ZOMBIE
(dead, but not cleaned up)
          ↓
Parent calls wait()
          ↓
Zombie is cleaned up ✅
```

### Demonstrating a Zombie

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        printf("Parent PID: %d, child PID: %d\n", getpid(), pid);
        sleep(10);           // parent sleeps — does NOT call wait()
        printf("Parent done.\n");
        wait(NULL);
        printf("Child process has been finished as well!");
    }
    else if (pid == 0) {
        printf("Child PID: %d working...\n", getpid());
        sleep(5);            // child works for 5 seconds
        printf("Child finished.\n");
        // child exits here — becomes ZOMBIE for 5 seconds
        // until parent wakes up and calls wait()
    }
}
```

During the window between when the child exits (second 5) and when the parent calls `wait()` (second 10), the child is a zombie. On Linux you can see it with:

```bash
ps aux | grep myprogram
# shows: [myprogram] <defunct>
```

The word `<defunct>` means zombie.

---

## 7. Orphan Processes

The opposite of a zombie: what if the **parent** dies before the child?

```
Parent dies while child is still running
               ↓
Child is now an ORPHAN
(has no parent to collect it)
```

### The init Process Adopts Orphans

Every Unix system has a special process with **PID = 1** called `init` (or `launchd` on macOS, `systemd` on modern Linux). It never dies. When a parent process dies, the OS automatically reassigns all its orphaned children to `init`:

```
BEFORE parent dies:           AFTER parent dies:

init (PID 1)                  init (PID 1)
│                             │
└── parent (PID 100)          └── child (PID 101)  ← adopted!
        │
        └── child (PID 101)
```

### Demonstrating an Orphan

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        printf("Parent PID:%d dying now!\n", getpid());
        // parent exits immediately
    }
    else if (pid == 0) {
        printf("Child PID:%d, my parent is: %d\n", getpid(), getppid());
        sleep(2);
        // by now parent is dead — getppid() returns 1
        printf("Child PID:%d, my parent is now: %d\n", getpid(), getppid());
        sleep(25);
    }
}
```

Output:

```
Parent PID:100 dying now!
Child PID:101, my parent is: 100
Child PID:101, my parent is now: 1    ← adopted by init!
```

### Zombie vs Orphan Summary

| | Zombie | Orphan |
|---|---|---|
| **Who dies first?** | Child | Parent |
| **The problem** | Child dead, not cleaned up | Child has no parent |
| **Solution** | Parent calls `wait()` | OS adopts it (init) |
| **Dangerous?** | Yes — wastes OS resources | No — OS handles it cleanly |

---

## 8. Waiting for Children — `wait()` and `waitpid()`

`wait()` is how a parent collects a finished child, preventing zombies and getting the child's result.

### Basic wait()

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        printf("Parent waiting...\n");
        wait(NULL);     // blocks here until ANY child finishes
        printf("Child finished! Parent continues.\n");
    }
    else if (pid == 0) {
        printf("Child working...\n");
        sleep(2);
        printf("Child done.\n");
    }

    return 0;
}
```

Output:

```
Parent waiting...
Child working...
Child done.
Child finished! Parent continues.
```

### The Exit Status — How Children Communicate Results

A child can send a number back to the parent when it exits:

```c
exit(42);    // child sends 42
```

The parent collects it, but needs to unpack it — because `wait()` stores more than just the exit code inside `status`. It packs the exit code, signal information, and other data together into one integer. `WEXITSTATUS()` unpacks it:

```c
int status;
wait(&status);
int code = WEXITSTATUS(status);   // extract the exit code
```

Think of it like a package:

```
Child sends:   exit(1)
                  ↓
OS wraps it:   ┌─────────────────────────┐
               │ exit code: 1            │
               │ died normally: yes      │
               │ signal number: 0        │
               └─────────────────────────┘
                  ↓
Parent receives raw package → status = 256
WEXITSTATUS unpacks it      → code = 1 ✅
```

### Full Example — Different Jobs, Exit Status

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int job1() {
    int sum = 0;
    for (int i = 0; i < 100; i++) sum += i;
    return sum;
}

int job2() {
    int product = 1;
    for (int i = 1; i <= 5; i++) product *= i;
    return product;
}

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        // PARENT does job1
        printf("Parent executing job1...\n");
        int result1 = job1();
        printf("job1 result: %d\n", result1);

        // collect child's exit status
        int status;
        wait(&status);
        int code = WEXITSTATUS(status);

        if (code == 0)
            printf("Child succeeded (product was 120).\n");
        else
            printf("Child failed (product was not 120).\n");
    }
    else if (pid == 0) {
        // CHILD does job2
        printf("Child executing job2...\n");
        int result2 = job2();
        printf("job2 result: %d\n", result2);

        if (result2 == 120) exit(0);   // success
        else                exit(1);   // failure
    }

    return 0;
}
```

By convention:
- `exit(0)` means **success**
- `exit(1)` means **failure**

### waitpid() — Waiting for a Specific Child

`wait()` collects whoever finishes first. `waitpid()` waits for a **specific** child by PID:

```c
waitpid(pids[i], &status, 0);   // wait specifically for child i
```

`wait()` returns the PID of whoever finished, which you can use to identify which child it was:

```c
pid_t finished_pid = wait(&status);
// match finished_pid against your pids[] array to find which child it was
```

### Other Useful Macros

```c
WIFEXITED(status)    // did child exit normally? (true/false)
WEXITSTATUS(status)  // what exit code did it give?
WIFSIGNALED(status)  // did child die from a signal?
WTERMSIG(status)     // which signal killed it?
```

In defensive programs, always check `WIFEXITED` before `WEXITSTATUS`:

```c
if (WIFEXITED(status)) {
    int code = WEXITSTATUS(status);
    printf("Child exited normally with code: %d\n", code);
} else {
    printf("Child crashed or was killed!\n");
}
```

---

## 9. Multiple Children — The Create Loop and Wait Loop Pattern

The most important pattern in process-based parallel programming is running multiple children simultaneously. The key insight is that creating and waiting must happen in **separate loops**.

### The Wrong Way — Sequential

```c
for (int i = 0; i < 3; i++) {
    fork();
    wait(NULL);   // ← WRONG: waits for each child before creating the next
}
```

This is like a restaurant chef hiring one kitchen assistant, watching them until they finish, then hiring the next one. Nobody works in parallel.

```
Sequential (wrong):
Child 0: [====]
Child 1:        [====]    ← waits for 0 to finish first
Child 2:               [====]
Total time = sum of all times. SLOW.
```

### The Right Way — Two Separate Loops

```c
pid_t pids[3];

// CREATE LOOP — spawn all children, don't wait
for (int i = 0; i < 3; i++) {
    pids[i] = fork();
    if (pids[i] == 0) {
        // child does its job and exits immediately
        // exit(0) is critical — otherwise child re-enters the loop!
        exit(0);
    }
}

// WAIT LOOP — collect all children
for (int i = 0; i < 3; i++) {
    int status;
    wait(&status);
}
```

This is like the chef giving all three assistants their jobs at once, then sitting down and waiting for whoever finishes first.

```
Parallel (correct):
Child 0: [====]
Child 1: [====]    ← all three run at the same time!
Child 2: [====]
Total time = time of the SLOWEST one. FAST.
```

### Why exit() Inside the Child Is Critical

Without `exit()`, each child re-enters the for loop and forks again, creating an exponentially growing tree of processes:

```
WITHOUT exit():               WITH exit():
i=0: Parent forks C0          i=0: Parent forks C0
i=1: Parent forks C1               C0 exits immediately ✅
     C0 also forks C2         i=1: Parent forks C1
i=2: Parent forks C3               C1 exits immediately ✅
     C0, C1, C2 all fork...   i=2: Parent forks C2
     CHAOS — exponential!          C2 exits immediately ✅
                               Clean: one parent, three children
```

### Full Example — Three Children, Different Jobs, Parallel

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int job_one() {
    int sum = 0;
    for (int i = 0; i < 10; i++) sum += i;
    return sum;   // 45
}

int job_two() {
    int product = 1;
    for (int i = 1; i <= 5; i++) product *= i;
    return product;   // 120
}

int job_three() {
    int difference = 100;
    for (int i = 1; i <= 10; i++) difference -= i;
    return difference;   // 45
}

int main() {
    pid_t pids[3];

    // CREATE LOOP
    for (int i = 0; i < 3; i++) {
        pids[i] = fork();

        if (pids[i] == 0) {
            sleep(rand() % 3);   // simulate different speeds
            printf("Child %d (PID:%d) starting...\n", i, getpid());

            int result;
            if      (i == 0) result = job_one();
            else if (i == 1) result = job_two();
            else             result = job_three();

            // report success or failure via exit code
            if      (i == 0 && result == 45)  exit(0);
            else if (i == 1 && result == 120) exit(0);
            else if (i == 2 && result == 45)  exit(0);
            else                              exit(1);
        }
        else if (pids[i] < 0) {
            printf("Fork failed\n");
            exit(1);
        }
        // parent does NOT wait here — just continues to next iteration
    }

    // WAIT LOOP — collect whoever finishes first
    for (int i = 0; i < 3; i++) {
        int status;
        pid_t finished_pid = wait(&status);
        int code = WEXITSTATUS(status);

        // figure out which child number finished
        int child_number = -1;
        for (int j = 0; j < 3; j++) {
            if (pids[j] == finished_pid) child_number = j;
        }

        if (code == 0)
            printf("Child %d finished successfully.\n", child_number);
        else
            printf("Child %d finished with failure.\n", child_number);
    }

    printf("Parent done. All children finished.\n");
    return 0;
}
```

Because of `sleep(rand() % 3)`, the output order is non-deterministic — a different child might finish first on each run:

```
Run 1:                          Run 2:
Child 1 finished successfully.  Child 0 finished successfully.
Child 0 finished successfully.  Child 2 finished successfully.
Child 2 finished successfully.  Child 1 finished successfully.
```

This is the nature of parallel execution.

---

## 10. Process Trees

Children can fork their own children, creating a tree of processes. Each node in the tree is a process, and each is responsible for waiting for its own direct children.

```c
int main() {
    pid_t pid1 = fork();   // parent creates child 1

    if (pid1 == 0) {
        pid_t pid2 = fork();   // child 1 creates child 2

        if (pid2 == 0) {
            // child 2 does its work
            exit(0);
        }
        wait(NULL);   // child 1 waits for child 2
        exit(0);      // child 1 exits
    }

    wait(NULL);   // parent waits for child 1
    return 0;
}
```

The resulting tree:

```
PARENT (runs job_product)
    │
    └── CHILD 1 (runs job_sum)
              │
              └── CHILD 2 (runs job_difference)
```

Each level waits for the one below it before reporting back up.

---

## 11. The Complete Picture — Everything in One Place

### Process lifecycle

```
program on disk
      ↓
OS loads it → PROCESS BORN (fork() or program launch)
      ↓
process runs
      ↓
process calls exit() → PROCESS DIES → becomes ZOMBIE
      ↓
parent calls wait() → ZOMBIE CLEANED UP ✅
```

### Key functions reference

| Function | Purpose |
|---|---|
| `getpid()` | Get my own PID |
| `getppid()` | Get my parent's PID |
| `fork()` | Create a child process |
| `wait(&status)` | Wait for any child, get PID back |
| `waitpid(pid, &status, 0)` | Wait for a specific child |
| `exit(code)` | Exit with a status code |
| `WEXITSTATUS(status)` | Unpack exit code from raw status |
| `WIFEXITED(status)` | Check if child exited normally |

### The parallel to threads

Every concept here maps directly to pthreads:

```
PROCESSES:                    THREADS:
fork()          →             pthread_create()
wait()          →             pthread_join()
waitpid()       →             pthread_join(t[i])
exit(code)      →             return (void*)value
WEXITSTATUS()   →             cast (void*) result
pid_t pids[N]   →             pthread_t t[N]
create loop     →             create loop
wait loop       →             join loop
```

The thinking is identical. Only the function names are different.

---

## 12. Processes vs Threads — When to Use Which

| | Processes | Threads |
|---|---|---|
| **Memory** | Separate (isolated) | Shared |
| **Communication** | Hard (need IPC) | Easy (shared variables) |
| **Safety** | Safe — no interference | Dangerous — race conditions |
| **Speed** | Slower to create | Faster to create |
| **Crash isolation** | One crash doesn't affect others | One crash kills all |
| **Used in** | MPI, web servers, shells | pthreads, OpenMP, CUDA |

Processes are the right tool when you need **isolation and safety**. Threads are the right tool when you need **speed and shared data**. Understanding both — and understanding why they differ — is what makes parallel programs correct.