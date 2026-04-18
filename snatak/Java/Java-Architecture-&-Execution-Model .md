# ☕ Java Architecture & Execution Model (Spring Boot Salary API)

<p align="center">
  <img width="120" src="https://cdn-icons-png.flaticon.com/512/226/226777.png" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Java-17-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/SpringBoot-3.x-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Maven-Build-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Microservice-SalaryAPI-success?style=for-the-badge" />
  <img src="https://img.shields.io/badge/DevOps-Ready-purple?style=for-the-badge" />
</p>

---

## 📑 Table of Contents

* [Introduction](#-introduction)
* [Architecture](#-architecture)
* [Java Execution Flow](#-java-execution-flow)
* [JVM vs JRE vs JDK (In Depth)](#-jvm-vs-jre-vs-jdk-in-depth)
* [Spring Boot Layered Architecture](#-spring-boot-layered-architecture)
* [Salary API — Real Mapping](#-salary-api--real-mapping)
* [DevOps & Runtime Perspective](#-devops--runtime-perspective)
* [End-to-End Lifecycle](#-end-to-end-lifecycle)
* [Interview Questions (Detailed Answers)](#-interview-questions-detailed-answers)
* [References](#-references)

---

## 📌 Introduction

If you strip Java down to its core idea, it is simply this: **you write code once, and it runs anywhere**. That promise is not magic—it is the result of a carefully designed execution model involving **bytecode and the JVM**.

In real-world systems like your **Spring Boot salary-api**, this model directly affects how your application is built, packaged, deployed, and run in production.

This README walks through everything step by step, using both **simple explanations** and **real project mapping**, so you don’t just memorize concepts—you actually understand how they work in practice.

---

## 🖼️ Architecture 

<p align="center">
  <img width="600" height="auto" alt="ChatGPT Image Apr 18, 2026, 09_42_22 AM" src="https://github.com/user-attachments/assets/471c5a11-6a0f-4bc0-9528-af5f5304e22f" />
</p>

---

## 🔄 Java Execution Flow

```
.java → javac → .class → JVM → Machine Code → Output
```

### 💡 What’s actually happening here?

Let’s break this down in a way that actually makes sense.

### 1. Source Code (`.java`)

You write Java code in a file like:

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello Mukesh");
    }
}
```

This is human-readable. The machine cannot understand this directly.

---

### 2. Compilation (`javac`)

```bash
javac Hello.java
```

The compiler checks:

* Syntax errors
* Type safety
* Structure of code

Then it converts your code into **bytecode**.

---

### 3. Bytecode (`.class`)

This is the most important concept.

* It is **not machine code**
* It is **not human readable**
* It is **platform-independent**

👉 This is the reason Java works everywhere.

---

### 4. JVM Execution

```bash
java Hello
```

Now JVM takes over:

* Loads the class
* Verifies it (security check)
* Converts it into machine code (using JIT)
* Executes it

---

### 5. Output

Finally, your program runs on the system.

---

## 🧩 JVM vs JRE vs JDK (In Depth)

This is where most confusion happens, so let’s make it crystal clear.

### 📊 Simple Comparison

| Component | What it really means                  |
| --------- | ------------------------------------- |
| JVM       | The engine that runs your program     |
| JRE       | Everything needed to run Java         |
| JDK       | Everything needed to build + run Java |

---

### 🔗 Relationship

```
JDK = JRE + Tools
JRE = JVM + Libraries
JVM = Execution Engine
```

---

### 🧠 Think of it like this:

* JVM = Engine of a car
* JRE = Engine + Fuel + Basic parts
* JDK = Full workshop (tools to build the car)

---

### 🔍 Inside JVM (Simplified)

JVM is not just one thing—it has multiple parts:

* **ClassLoader** → loads `.class` files
* **Memory Areas** → Heap, Stack
* **Execution Engine** → runs code
* **Garbage Collector** → cleans memory automatically

👉 This is why Java handles memory better than languages like C/C++.

---

## 🏗️ Spring Boot Layered Architecture

```
Client → Controller → Service → Repository → Database
```

This pattern is used in your salary-api.

### Why this structure exists?

Because mixing everything in one place becomes messy.

---

### Controller

* Entry point of API
* Handles HTTP requests
* Example: `/salary/create`

---

### Service

* Contains business logic
* Example: salary calculation

---

### Repository

* Talks to database
* Executes queries

---

### Model

* Represents table structure

---

## 🚀 Salary API — Real Mapping

### 📁 Project Structure

```
salary-api/
 ├── controller/
 ├── service/
 ├── repository/
 ├── model/
 ├── config/
 ├── pom.xml
```

---

### 🔄 Real Flow

#### Step 1 — Write Code

```java
@RestController
public class SpringDataController {}
```

---

#### Step 2 — Build

```bash
./mvnw clean install
```

What happens:

* Java code compiled
* Dependencies downloaded
* JAR file created

---

#### Step 3 — Output

```
target/salary-0.1.0-RELEASE.jar
```

---

#### Step 4 — Run

```bash
java -jar target/salary-0.1.0-RELEASE.jar
```

What happens internally:

* JVM starts
* Spring Boot loads
* Server starts on port 8082

---

#### Step 5 — Verify

```bash
curl http://localhost:8082/actuator/health
```

```json
{"status":"UP"}
```

---

## 🐳 DevOps & Runtime Perspective

### Build vs Run (Very Important)

| Stage | What is used |
| ----- | ------------ |
| Build | JDK          |
| Run   | JVM          |

👉 In production, you usually DO NOT use full JDK.

---

### Docker Example

```dockerfile
FROM eclipse-temurin:17-jre
COPY target/app.jar app.jar
CMD ["java", "-jar", "app.jar"]
```

---

## 🔁 End-to-End Lifecycle

```
Developer → Build → JAR → JVM → API → Response
```

In real DevOps:

```
Code → Git → CI/CD → Build → Docker → Deploy → Run
```

---

## 🎯 Interview Questions (Detailed Answers)

### 1. What is JVM?

JVM is the runtime engine that executes Java bytecode. It ensures that the same program runs on different systems without changes.

---

### 2. Difference between JRE and JDK?

JRE is used to run Java programs, while JDK is used to develop them. JDK includes tools like compiler, whereas JRE does not.

---

### 3. Why Java is platform independent?

Because it uses bytecode, which runs on JVM instead of directly on OS.

---

### 4. What is JIT?

JIT compiler improves performance by converting frequently used code into machine code at runtime.

---

### 5. What happens when you run a JAR?

JVM loads classes, initializes Spring Boot, starts server, and exposes APIs.

---

### 6. Can Java run without JDK?

Yes, it can run using JRE/JVM.

---

### 7. JVM memory structure?

Heap stores objects, stack stores method calls, and GC cleans memory automatically.

---

### 8. How Spring Boot works?

It auto-configures everything and starts an embedded server.

---

### 9. Java in Docker?

Packaged as JAR, runs inside container using JVM.

---

### 10. Compile vs Runtime?

Compile checks code, runtime executes it.

---

## 🔗 References

* [https://docs.oracle.com/javase/](https://docs.oracle.com/javase/)
* [https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)

---

## 🏁 Final Summary

Java works using a **compile once, run anywhere model** powered by JVM. In real-world systems like Spring Boot microservices, this translates into a clean build-run lifecycle and scalable architecture.

---

**⭐ Clean, practical, and interview-ready documentation**
