## 🐧 Types of Processes in Linux

In Linux, a **process** is a running instance of a program 🧠. Processes are categorized based on how they are created, executed, and managed by the kernel.

---

## 🔹 1. Foreground Process

📌 A process that runs in the terminal and occupies it until completion.

**Example:**

```bash
ls -l
```

➡️ The command runs and the shell waits until it finishes.

---

## 🔹 2. Background Process

📌 A process that runs in the background and does not block the terminal.

**Example:**

```bash
sleep 100 &
```

➡️ The terminal is free while the process continues to run.

---

## 🔹 3. Daemon Process

📌 A long-running background process that starts at boot time and runs without user interaction 🚀.

**Characteristics:**

* No controlling terminal
* Usually ends with `d`

**Examples:**

* `sshd` 🔐
* `cron` ⏰
* `systemd`

Check daemon process:

```bash
ps -ef | grep sshd
```

---

## 🔹 4. Parent Process

📌 A process that creates one or more child processes.

**Example:**

* `bash` shell is the parent of commands you execute

```bash
ps -ef | grep bash
```

---

## 🔹 5. Child Process

📌 A process created by another process using `fork()` system call.

**Example:**

* When you run `ls`, it becomes a child of the shell (`bash`).

---

## 🔹 6. Zombie Process 🧟

📌 A process that has completed execution but still has an entry in the process table because the parent has not collected its exit status.

**Characteristics:**

* Process state: `Z`
* Consumes no CPU or memory

**Example:**

```bash
ps aux | grep Z
```

➡️ Fixed when parent process exits or is restarted.

---

## 🔹 7. Orphan Process

📌 A process whose parent has terminated, but the child is still running.

**Behavior:**

* Adopted by `init` or `systemd` (PID 1)

**Example:**

* Parent shell closes while background job is running.

---

---

## 🧠 Process States in Linux

Linux process states describe the **current condition** of a process at a given time.

---

## 🔸 1. Running (R)

📌 The process is either executing on the CPU or ready to run.

**Example:**

```bash
top
```

➡️ Processes using CPU are in `R` state.

---

## 🔸 2. Sleeping (S)

📌 The process is waiting for an event or resource (interruptible sleep).

**Example:**

```bash
sleep 50
```

➡️ Waiting for time to expire ⏳.

---

## 🔸 3. Uninterruptible Sleep (D)

📌 The process is waiting for I/O operations and cannot be interrupted.

**Common cause:**

* Disk or network I/O issues 💽

**Example:**

* NFS mount hang

Check:

```bash
ps aux | grep D
```

---

## 🔸 4. Stopped (T)

📌 The process is paused, usually by a signal.

**Example:**

```bash
Ctrl + Z
```

➡️ Process is stopped but still in memory.

---

## 🔸 5. Zombie (Z) 🧟

📌 The process is terminated but not fully cleaned up.

**Example:**

* Parent didn’t call `wait()`

Check:

```bash
ps -el | grep Z
```

---

## 🔸 6. Dead (X)

📌 Process is fully terminated and removed from process table.

➡️ Rarely visible to users.

---

## 📊 Quick Summary Table

| Type / State | Meaning                  |
| ------------ | ------------------------ |
| R            | Running or Ready         |
| S            | Sleeping (Interruptible) |
| D            | Uninterruptible Sleep    |
| T            | Stopped                  |
| Z            | Zombie                   |
| X            | Dead                     |

---

## 🎯 Interview Tip

🗣️ **Zombie vs Orphan**

* Zombie ❌ → Parent alive, child exited
* Orphan ✔️ → Parent exited, child alive

---

## 🚀 DevOps Relevance

* High `D` state → Disk or NFS issue ⚠️
* Many `Z` processes → Buggy application 🐞
* High `R` state count → CPU bottleneck 🔥

Linux process knowledge is critical for **production troubleshooting and performance tuning** 💡.
