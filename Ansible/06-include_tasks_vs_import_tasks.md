# 🔀 include_tasks vs import_tasks (Ansible)

Both `include_tasks` and `import_tasks` are used to **reuse task files**, but they behave **very differently**. This difference is a **classic interview question**.

---

## 🧠 Core Difference (One Line)

> **`import_tasks` is static (decided at playbook parse time), while `include_tasks` is dynamic (decided at runtime).**

---

## ⚙️ How `import_tasks` Works (Static)

`import_tasks` is processed **when Ansible parses the playbook**, before execution starts.

### Example

```yaml
- hosts: web
  tasks:
    - import_tasks: install.yml
```

### What happens internally

* Tasks from `install.yml` are **merged into the playbook**
* Ansible knows **all tasks in advance**
* Conditions are applied **per task**, not to the import itself

### Key traits

* 📦 Tasks are loaded **once**
* 🧠 Execution plan is fixed
* 🚫 Cannot use variables to decide whether to import

---

## 🔄 How `include_tasks` Works (Dynamic)

`include_tasks` is processed **during execution**, at runtime.

### Example

```yaml
- hosts: web
  tasks:
    - include_tasks: install.yml
      when: ansible_os_family == "Debian"
```

### What happens internally

* Ansible reaches this task during execution
* Condition is evaluated
* Tasks are included **only if the condition is true**

### Key traits

* 🔁 Tasks can be included **multiple times**
* 🎯 Conditions apply to the **entire include**
* ✅ Variables and facts fully supported

---

## 📊 Side-by-Side Comparison

| Aspect              | import_tasks | include_tasks |
| ------------------- | ------------ | ------------- |
| Evaluation time     | Parse time   | Runtime       |
| Static / Dynamic    | Static       | Dynamic       |
| Conditional include | ❌ No         | ✅ Yes         |
| Loop support        | ❌ No         | ✅ Yes         |
| Uses facts          | ❌ No         | ✅ Yes         |
| Execution plan      | Fixed        | Flexible      |

---

## 🔍 Real DevOps Example

### Scenario

Install packages differently based on OS.

### Using `include_tasks` (Correct)

```yaml
- include_tasks: debian.yml
  when: ansible_os_family == "Debian"

- include_tasks: redhat.yml
  when: ansible_os_family == "RedHat"
```

### Why not `import_tasks`?

Because OS facts are available **only at runtime**.

---

## ⚠️ Common Mistake

```yaml
- import_tasks: install.yml
  when: ansible_os_family == "Debian"
```

❌ The `when` is applied to **each task inside**, not to the import itself.

---

## 🧠 When to Use What

### ✅ Use `import_tasks` when:

* Task structure is fixed
* No runtime conditions
* You want faster parsing

### ✅ Use `include_tasks` when:

* Tasks depend on facts or variables
* Conditional execution is required
* Tasks need to run in loops

---

## 🎯 Interview-Perfect Answer

> **“`import_tasks` is static and processed at playbook parse time, while `include_tasks` is dynamic and evaluated at runtime, making `include_tasks` suitable for conditional and fact-based task execution.”**
