## 🎤 Mock Interview Question 1

### ❓ Question:

Can you explain how execution works in Go and how it is different from Java and Python?

---

### 🧑‍💻 Answer:

In Go, execution follows a **direct compilation model**, whereas Java uses a **JVM-based execution model**, and Python follows an **interpreted execution model**.

---

### 🐹 Go Execution

When we run:

```bash
go build main.go
```

The Go compiler converts the source code (`.go` files) directly into a **native binary executable** which runs directly on the operating system.

```mermaid
flowchart LR
    A[.go Source Code] --> B[Go Compiler]
    B --> C[Binary Executable]
    C --> D[Operating System]
    D --> E[Output]
```

👉 Go has **no JVM and no intermediate layer**.

---

### ☕ Java Execution

Java follows a two-step execution process.

```bash
javac Hello.java
java Hello
```

```mermaid
flowchart LR
    A[.java Source Code] --> B[javac Compiler]
    B --> C[Bytecode .class]
    C --> JVM
    %% JVM as a vertical boxed layer
   

    subgraph JVM [JVM - Java Virtual Machine]
        direction TB
        J1[Class Loader]
        J2[JIT -> Just-In-Time Compiler]
        J3[Runtime Memory & Garbage Collection]
        J1 --> J2 --> J3
    end

    JVM --> E[Operating System]
    E --> F[Output]

%% Darkest fills
    style A fill:#1e88e5,stroke:#000,stroke-width:2px
    style B fill:#2e7d32,stroke:#000,stroke-width:2px
    style C fill:#ef6c00,stroke:#000,stroke-width:2px
    style J1 fill:#c2185b,stroke:#000,stroke-width:2px
    style J2 fill:#c2185b,stroke:#000,stroke-width:2px
    style J3 fill:#c2185b,stroke:#000,stroke-width:2px
    style E fill:#512da8,stroke:#000,stroke-width:2px
```

👉 In Java, bytecode enters the **JVM (Java Virtual Machine)** where:

* **Class Loader** loads `.class` files
* **JIT (Just-In-Time) Compiler** converts bytecode into native machine code at runtime
* **Runtime** handles memory management and garbage collection

---

### 🐍 Python Execution

```bash
python app.py
```

```mermaid
flowchart LR
    A[.py Source Code] --> B[Python Interpreter]
    B --> C[Bytecode .pyc]
    C --> D[Python VM]
    D --> E[Output]
```

👉 Python uses an interpreter and is generally slower.

---
