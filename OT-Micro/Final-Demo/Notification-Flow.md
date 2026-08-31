# OT-Microservices — Notification Flow 📨

> **Presentation Cheat Sheet**
>
> **Remember:** `Frontend → APIs → ScyllaDB → Notification API → Elasticsearch → SMTP → Email`

## 1. Big Picture

The Notification API has **two jobs**:

1. Sync data: **ScyllaDB → Elasticsearch**
2. Send mail: **Elasticsearch → SMTP → Employee**

```text
                 FRONTEND
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Employee API          Salary API
          │                   │
          ▼                   ▼
     ScyllaDB             ScyllaDB
 employee_info          employee_salary
          │                   │
          └─────────┬─────────┘
                    ▼
             Notification API
                    │
             ScyllaDB → ES
                    ▼
           Elasticsearch
          employee_index
                    │
          pending notifications
                    ▼
                 SMTP
                    │
                    ▼
              Employee Email
```

## 2. Remember the Flow in 7 Steps

### 1 — Employee is created

Frontend calls:

```text
POST /api/v1/employee/create
```

Employee API writes employee details to:

```text
ScyllaDB → employee_info
```

Important field: `email`

### 2 — Salary is created

Frontend calls:

```text
POST /api/v1/salary/create/record
```

Salary API writes:

```text
ScyllaDB → employee_salary
```

Important fields:

`id, name, salary, process_date, status`

### 3 — Frontend triggers Notification API

After employee + salary creation:

```text
POST /api/v1/notification/send/all
```

⚠️ The frontend **does not send the email data here**.

It only tells Notification API:

> "Process the pending notifications."

### 4 — Notification API reads ScyllaDB

It reads:

```text
employee_salary
       +
employee_info
```

It gets:

```text
salary
process_date
status
name
email
designation
```

Then it combines them into one notification document.

### 5 — Notification API writes Elasticsearch

Default index:

```text
employee_index
```

The code builds:

```python
document = {
    "employee_id": salary.id,
    "name": employee.name,
    "email_id": employee.email,
    "designation": employee.designation,
    "salary": salary.salary,
    "process_date": str(salary.process_date),
    "status": salary.status,
    "notified": notified
}
```

Then:

```python
es_client.index(
    index=es_index,
    id=employee.email,
    body=document
)
```

### ⭐ Key point

**ScyllaDB is the source of truth.**

**Elasticsearch is the notification processing/read model.**

### 6 — Find pending emails

Notification API searches Elasticsearch for:

```text
notified != true
```

Example:

```text
EMP001 → notified=true   → ❌ Skip
EMP002 → notified=false  → ✅ Send
EMP003 → notified=false  → ✅ Send
```

### 7 — Send email and mark it sent

Notification API calls:

```text
send_mail(email)
```

SMTP:

```text
smtp.gmail.com:587
STARTTLS
```

After successful sending:

```text
Elasticsearch
      │
      ▼
notified = true
```

This prevents duplicate notifications.

---

# 🧠 Interview Memory Trick

Remember:

## **C → S → T → R → I → S → U**

| Letter | Meaning |
|---|---|
| **C** | Create employee |
| **S** | Salary created |
| **T** | Trigger Notification API |
| **R** | Read ScyllaDB |
| **I** | Index into Elasticsearch |
| **S** | Send SMTP email |
| **U** | Update `notified=true` |

Say it mentally:

> **Create → Salary → Trigger → Read → Index → Send → Update**

---

# 🔍 Where is each thing?

| What | Where |
|---|---|
| Employee data | `employee_info` in ScyllaDB |
| Salary data | `employee_salary` in ScyllaDB |
| ES index | `employee_index` |
| ES document ID | Employee email |
| Email recipient | ES field `email_id` |
| Email server | `smtp.gmail.com:587` |
| Sent flag | ES field `notified` |

---

# 📂 Code Locations

### Frontend

```text
Frontend-main/src/EmployeeForm.js
```

Triggers:

```text
/api/v1/notification/send/all
```

### Notification API

```text
Notification-main/notification_api.py
```

Main functions:

```text
send_all_notifications()
        ↓
sync_scylla_to_es()
        ↓
process_pending_notifications()
        ↓
send_mail()
```

### Configuration

```text
Notification-main/config.yaml
```

Contains Elasticsearch and SMTP configuration.

---

# 🎤 30-Second Presentation Answer

> "The frontend first creates the employee and salary records through the Employee and Salary APIs, and both are stored in ScyllaDB. Once both operations succeed, the frontend triggers the Notification API using `/api/v1/notification/send/all`. The Notification API reads the salary and employee information from ScyllaDB, combines them into a notification document, and stores it in the `employee_index` Elasticsearch index. It then searches Elasticsearch for records where `notified` is not true, sends the email using Gmail SMTP on port 587 with STARTTLS, and after successful delivery updates the document to `notified=true` to prevent duplicate emails."

---

# ❓ Quick Self-Test

Try answering these **without looking above**.

### Q1. Does Frontend directly send the email?

<details>
<summary>👉 Reveal answer</summary>

**No.** Frontend only triggers:

`POST /api/v1/notification/send/all`

</details>

### Q2. Where is the employee email originally stored?

<details>
<summary>👉 Reveal answer</summary>

**ScyllaDB → `employee_info`**

</details>

### Q3. Where does Notification API get salary information?

<details>
<summary>👉 Reveal answer</summary>

**ScyllaDB → `employee_salary`**

</details>

### Q4. Who writes the notification document to Elasticsearch?

<details>
<summary>👉 Reveal answer</summary>

**Notification API**, using `sync_scylla_to_es()`.

</details>

### Q5. What is the Elasticsearch index?

<details>
<summary>👉 Reveal answer</summary>

`employee_index`

</details>

### Q6. How does it know which emails are pending?

<details>
<summary>👉 Reveal answer</summary>

It searches for documents where:

`notified != true`

</details>

### Q7. What happens after a successful email?

<details>
<summary>👉 Reveal answer</summary>

Elasticsearch is updated:

`notified = true`

</details>

### Q8. What is the source of truth?

<details>
<summary>👉 Reveal answer</summary>

**ScyllaDB.**

Elasticsearch is the notification processing/read model.

</details>

---

# 🚨 One Important Architecture Point

Do **not** say:

> "Salary API writes the data to Elasticsearch."

❌ Incorrect for this project.

Say:

> **"Notification API synchronizes the required data from ScyllaDB into Elasticsearch."**

That matches the implementation.

---

# 🏁 Final Mental Picture

```text
USER
 │
 ▼
FRONTEND
 │
 ├── Employee API ──→ ScyllaDB.employee_info
 │
 ├── Salary API ────→ ScyllaDB.employee_salary
 │
 └── Notification API
          │
          ▼
      READ SCYLLA
          │
          ▼
      WRITE ES
          │
          ▼
    employee_index
          │
     notified=false
          │
          ▼
      SEND SMTP
          │
          ▼
       EMAIL 📧
          │
          ▼
     notified=true
```

> **If you remember only one thing:**
>
> ### `ScyllaDB = Source of Truth`
> ### `Elasticsearch = Notification Read/Processing Model`
> ### `SMTP = Actual Email Delivery`
