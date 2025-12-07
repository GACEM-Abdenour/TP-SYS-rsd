# Semaphore & Process Cheat Sheet

## 📑 Table of Contents
- [Keys & Identifiers](#keys--identifiers)
- [Semaphore Control](#semaphore-control)
- [Semaphore Operations](#semaphore-operations)
- [Standard Operations](#standard-operations)
- [Synchronization Patterns](#synchronization-patterns)
- [Cleanup](#-cleanup)
- [Process & Signal Quick Reference](#-process--signal-quick-reference)
- [Table of Matters (Quick Reference)](#-table-of-matters-quick-reference)

---


## Keys & Identifiers
- **`ftok(path, proj_id)`**
  - Generates a unique key (`key_t`) from a file path + project ID.
  - Use when you need a consistent identifier across multiple processes.

- **`semget(key, n, flags)`**
  - Creates or retrieves a semaphore set.
  - Returns a **semaphore identifier (`semid`)**.
  - Use when you need to **create a group of semaphores** or attach to an existing one.

---

## Semaphore Control
- **`semid`**
  - Identifier for a semaphore set (returned by `semget`).
  - Needed in almost every semaphore operation (`semctl`, `semop`).

- **`semctl(semid, semnum, cmd, arg)`**
  - Controls or queries a semaphore.
  - **Arguments:**
    - `semid` → ID of the semaphore set.
    - `semnum` → index of the semaphore in the set (ignored for some commands like `IPC_RMID`).
    - `cmd` → command to execute (`SETVAL`, `GETVAL`, `SETALL`, `GETALL`, `IPC_RMID`…).
    - `arg` → union `semun` (value or array depending on command).
  - **Common commands:**
    - `SETVAL` → initialize a semaphore.
    - `GETVAL` → read current value.
    - `SETALL` / `GETALL` → set/get values for all semaphores in a set.
    - `IPC_RMID` → destroy the semaphore set.
  - **Examples:**
    ```c
    union semun {
        int val;
        struct semid_ds *buf;
        unsigned short *array;
    } arg;

    // Initialize semaphore #0 to 1
    arg.val = 1;
    semctl(semid, 0, SETVAL, arg);

    // Read current value of semaphore #0
    int value = semctl(semid, 0, GETVAL);
    printf("Semaphore value = %d\n", value);

    // Destroy the semaphore set
    semctl(semid, 0, IPC_RMID);
    ```

---

## Semaphore Operations
- **`struct sembuf`**
  - This is an **existing system type** (declared in `<sys/sem.h>`).
  - You don’t define it yourself; you just create an instance and fill its fields.
  - Fields:
    - `sem_num`: which semaphore in the set.
    - `sem_op`: operation (`-1 = P`, `+1 = V`, `0 = wait until 0`).
    - `sem_flg`: options (usually `0`).
  - **Example:**
    ```c
    struct sembuf op;
    op.sem_num = 0;   // semaphore index
    op.sem_op  = -1;  // P operation (decrement)
    op.sem_flg = 0;   // default

    semop(semid, &op, 1); // apply the operation
    ```

- **`semop(semid, &operation, 1)`**
  - Applies the operation(s) defined in one or more `struct sembuf`.
  - Parameters:
    1. `semid` → semaphore set ID.
    2. `&operation` → pointer to your `struct sembuf` (or array of them).
    3. `1` → number of operations in the array.
  - Use when you need to **actually perform P, V, or Z**.

---

##  Standard Operations
- **P (wait / down)**
  - `sem_op = -1`
  - Decrements semaphore; blocks if result < 0.
  - Use to **enter critical section**.

- **V (signal / up)**
  - `sem_op = +1`
  - Increments semaphore; wakes waiting processes.
  - Use to **leave critical section**.

- **Z (zero wait)**
  - `sem_op = 0`
  - Blocks until semaphore value = 0.
  - Use to **synchronize until resource is free**.

---

##  Synchronization Patterns
- **Barrier**
  - All processes must reach the barrier before continuing.
  - Implemented with:
    - One semaphore as a **counter**.
    - One semaphore as a **mutex**.
    - One semaphore as a **queue**.
  - Use when you need **all processes to wait for each other**.

- **Mutual Exclusion (Mutex)**
  - Initialize semaphore to 1.
  - Use P before entering critical section, V after leaving.
  - Use when you need **only one process at a time**.

- **Producer–Consumer**
  - Shared memory + semaphores:
    - One semaphore for empty slots.
    - One semaphore for full slots.
    - One semaphore for mutual exclusion.
  - Use when processes **share a buffer**.

---

## 🧹 Cleanup
- **`semdestroy(semid)`**
  - Wrapper around `semctl(semid, 0, IPC_RMID)`.
  - Use when you need to **remove a semaphore set** after use.

---

# 🖥️ Process & Signal Quick Reference

- **`fork()`**
  - Creates a child process.
  - Use when you need parallel execution.

- **`getpid()` / `getppid()`**
  - Get process ID / parent process ID.
  - Use when identifying processes.

- **`wait()` / `waitpid()`**
  - Parent waits for child termination.
  - Use when synchronizing parent/child completion.

- **`kill(pid, signal)`**
  - Send a signal to a process.
  - Use for communication or termination.

- **Common Signals**
  - `SIGINT` → interrupt (Ctrl+C).
  - `SIGKILL` → force kill (`kill -9`).
  - `SIGUSR1` / `SIGUSR2` → user-defined signals.
  - Use signals for **process communication or control**.
