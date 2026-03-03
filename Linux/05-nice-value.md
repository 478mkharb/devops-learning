## 🐧 Nice Value in Linux (Process Priority)

The **nice value** controls the **CPU scheduling priority** of a process 🧠. It tells the Linux scheduler **how “nice” a process is to others** when competing for CPU time.

👉 Lower nice value = **higher priority**
👉 Higher nice value = **lower priority**

---

## 🎚️ Nice Value Range (VERY IMPORTANT)

```text
-20  -------------------->  +19
Highest priority            Lowest priority
```

* **-20** → Highest CPU priority 🚀
* **0** → Default priority (most processes)
* **+19** → Lowest CPU priority 🐢

📌 Normal users can increase nice value (lower priority)
📌 Only root can decrease nice value (increase priority)

---

## ⚙️ How Nice Value Works Internally

Linux uses the **Completely Fair Scheduler (CFS)**.

Nice value affects:

* How often a process gets CPU time
* How long it runs before being preempted

💡 Scheduler goal:

> Give more CPU time to processes with **lower nice values**.

---

## 🔍 Viewing Nice Value

Using `top`:

```bash
top
```

Look at columns:

* `NI` → Nice value
* `PR` → Priority calculated by kernel

Example:

```text
PID   USER   PR  NI   %CPU  COMMAND
1234  app    20   0   15.0  java
```

---

## ▶️ Starting a Process with Nice Value

```bash
nice -n 10 sleep 300
```

🔹 Starts `sleep` with **lower priority** (nice = +10)

Verify:

```bash
ps -o pid,ni,cmd -p <PID>
```

---

## 🔁 Changing Nice Value of Running Process

```bash
renice -n 5 -p 1234
```

➡️ Changes nice value of PID `1234` to `+5`

⚠️ Only root can set negative nice values:

```bash
sudo renice -n -10 -p 1234
```

---

## 📊 Nice vs Priority (PR)

| Field | Meaning                    |
| ----- | -------------------------- |
| NI    | User-defined niceness      |
| PR    | Kernel-calculated priority |

📌 Lower `PR` number = higher priority

---

## 🧪 Practical Examples (DevOps)

### 🔹 Example 1: Background Job

Run backup with low priority:

```bash
nice -n 15 tar -czf backup.tar.gz /data
```

➡️ Backup won’t slow production traffic 🛡️

---

### 🔹 Example 2: Production Application

Critical app needs more CPU:

```bash
sudo renice -n -5 -p <app_pid>
```

➡️ App gets CPU preference 🚀

---

## 🚨 Important Notes

* Nice value affects **CPU only**, not memory or I/O
* It does NOT guarantee CPU, only influences scheduling
* Overusing negative nice can starve other processes ⚠️

---

## 🎯 Interview One-Liners

**Q: What is nice value?**
Nice value defines process priority in Linux by influencing CPU scheduling.

**Q: Nice value range?**
From **-20 (highest priority)** to **+19 (lowest priority)**.

---

## 🚀 DevOps Tip

Use nice values to:

* Protect production workloads 🛡️
* Run maintenance jobs safely ⚙️
* Avoid CPU starvation incidents 💡

Nice value control is a **must-know skill for Linux performance tuning**.
