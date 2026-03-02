## 🐳 docker diff Command — Detailed Explanation with Examples

The `docker diff` command shows **filesystem-level changes** made inside a container compared to the **original image** it was created from.

In simple words:

> It tells you **what files were added, modified, or deleted** after the container started.

---

## 📌 Syntax

```bash
docker diff <container_id | container_name>
```

---

## 📂 What docker diff Tracks

`docker diff` tracks **only filesystem changes** inside the container.

### ✅ Tracked

* Files created
* Files modified
* Files deleted

### ❌ Not Tracked

* Environment variables
* Running processes
* Network changes
* Mounted volumes

---

## 🔍 Output Symbols Explained

Each line in the output starts with a letter:

| Symbol | Meaning | Description                   |
| ------ | ------- | ----------------------------- |
| A      | Added   | New file or directory created |
| C      | Changed | Existing file modified        |
| D      | Deleted | File or directory removed     |

---

## 🧪 Example 1: Added & Deleted Files

### Step 1: Run a container

```bash
docker run -it --name diff-demo ubuntu bash
```

### Step 2: Make changes inside container

```bash
touch /file1.txt
echo "hello" > /file2.txt
rm /etc/hostname
exit
```

### Step 3: Check changes

```bash
docker diff diff-demo
```

### Output

```text
A /file1.txt
A /file2.txt
D /etc/hostname
```

### Explanation

* `A /file1.txt` → New file added
* `A /file2.txt` → New file added
* `D /etc/hostname` → File deleted

---

## 🧪 Example 2: Modified File

### Modify an existing file

```bash
docker start diff-demo
docker exec -it diff-demo bash
echo "test" >> /etc/hosts
exit
```

### Run docker diff

```bash
docker diff diff-demo
```

### Output

```text
C /etc/hosts
```

### Explanation

* `/etc/hosts` existed in the image
* File content was modified

---

## 🚫 Important: Volumes Are Ignored

Changes inside volumes are **not shown** by `docker diff`.

### Example

```bash
docker run -it -v /data --name volume-test ubuntu bash
```

```bash
echo "hello" > /data/test.txt
exit
```

```bash
docker diff volume-test
```

### Output

```text
(no output)
```

### Reason

Volume data is stored **outside the container’s writable layer**, so Docker does not track it.

---

## 🎯 Real-World Use Cases

* Debug unexpected file changes inside containers
* Identify files that should be part of a Dockerfile
* Security auditing and intrusion detection
* Understanding container behavior before committing an image

---

## 📊 docker diff vs docker inspect

| Feature               | docker diff | docker inspect |
| --------------------- | ----------- | -------------- |
| Filesystem changes    | ✅           | ❌              |
| Metadata & config     | ❌           | ✅              |
| Environment variables | ❌           | ✅              |
| Volume changes        | ❌           | ❌              |

---

## ✅ Key Takeaways

* `docker diff` shows filesystem differences
* Works on running and stopped containers
* Displays changes using A / C / D
* Does not track volumes
* Useful for debugging and auditing containers
