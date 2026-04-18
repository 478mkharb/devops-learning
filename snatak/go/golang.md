# 🐹 Golang Architecture & Execution Model (Employee API)

<p align="center">
  <img width="120" src="https://go.dev/blog/go-brand/Go-Logo/PNG/Go-Logo_Blue.png" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Language-Go-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Framework-Gin-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Microservice-EmployeeAPI-success?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Architecture-REST-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Level-Interview%20Ready-purple?style=for-the-badge" />
</p>

---

# 📌 Overview

This README is a **deep, practical, and interview-focused guide** to understanding:

* How Go applications are structured
* How execution works internally
* How your **employee-api** actually runs
* How Go compares with Java and Python
* How request flows inside your microservice

This is not theory — this is **real system understanding**.

---

# 🧠 Concept Shift (Java → Go)

In Java, architecture is:

```
Controller → Service → Repository → DB
```

In Go, architecture is:

```
Packages → Functions → Responsibility
```

👉 There is **no strict layer enforcement**
👉 You design based on **purpose, not rules**

---

## 🔹 Concept Mapping

| Java          | Go Equivalent   |
| ------------- | --------------- |
| Class         | Struct          |
| Object        | Struct instance |
| Service Layer | Package         |
| Controller    | api/ package    |
| Repository    | client/ package |

---

# ⚙️ Execution Model 

## 🐹 Go Execution

```
.go → go build → binary → OS → Output
```

### Step-by-step explanation:

1. You write `.go` files
2. Go compiler compiles ALL packages
3. Generates a **native binary**
4. OS executes binary directly

👉 No JVM
👉 No interpreter
👉 No runtime dependency

---

## ☕ Java Execution

```
.java → javac → .class → JVM → Output
```

* Requires JVM
* Uses bytecode
* Extra abstraction layer

---

## 🐍 Python Execution

```
.py → Interpreter → Bytecode → Execution
```

* Interpreted
* Slower performance

---

## ⚔️ Comparison

| Feature     | Go      | Java   | Python      |
| ----------- | ------- | ------ | ----------- |
| Execution   | Native  | JVM    | Interpreter |
| Startup     | Instant | Slow   | Medium      |
| Deployment  | Binary  | JAR    | Script      |
| Performance | High    | Medium | Low         |

---

# 📁 Project Structure-Employee API

```
employee-api/
 ├── api/          → Request handlers
 ├── routes/       → Route definitions
 ├── middleware/   → Pre-processing logic
 ├── client/       → Database interaction
 ├── model/        → Data structures
 ├── config/       → Configuration (Viper)
 ├── migration/    → DB schema
 ├── main.go       → Entry point
```

---

# 🔥 Folder Responsibilities

## api/

* Handles HTTP requests
* Equivalent to Controller
* Returns JSON responses

---

## routes/

* Maps URL → handler
* Central routing logic

---

## middleware/

* Executes before handler
* Logging, auth, validation

---

## client/

* DB communication layer
* Talks to ScyllaDB, Redis

---

## model/

* Struct definitions
* Defines data format

---

## config/

* Reads configuration using Viper

---

## main.go

* Entry point of application
* Initializes everything

---

# 🌐 Request Flow

```
Client → Router → Middleware → Handler → Business Logic → DB → Response
```

---

## 🔍 Step-by-step Flow

1. Client sends request
2. Router matches endpoint
3. Middleware runs
4. Handler executes
5. Business logic runs
6. DB query executed
7. Response returned

---

## 💻 Example

```go
router.GET("/employee/:id", func(c *gin.Context) {
    id := c.Param("id")

    employee := map[string]string{
        "id": id,
        "name": "Mukesh",
    }

    c.JSON(200, employee)
})
```

---

# 🔧 Viper Configuration

## Why needed?

Hardcoding values is bad:

```go
port := 8080
```

Better approach:

```yaml
server:
  port: 8080
```

```go
port := viper.GetInt("server.port")
```

👉 Makes app flexible
👉 Supports environment configs

---

# 🔄 CRUD Operations

| Operation | Method | Example     |
| --------- | ------ | ----------- |
| Create    | POST   | /employee   |
| Read      | GET    | /employee/1 |
| Update    | PUT    | /employee/1 |
| Delete    | DELETE | /employee/1 |

---

# 🚀 What Happens When You Run?

```bash
go run main.go
```

### Internally:

1. Compiler reads all packages
2. Compiles code
3. Links dependencies
4. Creates temporary binary
5. Executes `main()`
6. Starts server

---

# 🎯 Why Go is Preferred

### 1. Performance

Compiled → faster execution

### 2. Simplicity

Less boilerplate

### 3. Concurrency

Goroutines (lightweight threads)

### 4. Deployment

Single binary (no dependency issues)

### 5. Fast Startup

No JVM overhead

---

# 🎯 Interview Questions

## Q1: What is Gin?

Gin is a lightweight web framework in Go used to build APIs efficiently. It provides routing, middleware support, and high performance.

---

## Q2: What replaces OOP in Go?

Go uses structs and packages instead of traditional OOP.

---

## Q3: Why Go is faster than Java?

Because it runs directly as a binary without JVM overhead.

---

## Q4: What happens in go run?

It compiles and executes code in one step.

---

## Q5: Why Go for microservices?

Because of speed, simplicity, and easy deployment.

---

# 🏁 Final Summary

Go is designed for **simplicity, performance, and scalability**. It eliminates unnecessary abstraction layers like JVM and provides a clean, efficient way to build microservices like your employee-api.

---

<p align="center">
  ⭐ Production-ready • Interview-ready • DevOps-friendly
</p>
