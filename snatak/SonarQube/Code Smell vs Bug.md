# Code Smell vs Bug

Static code analysis tools like SonarQube identify different types of issues in the source code. Two common categories are **Code Smells** and **Bugs**.

| Feature | Code Smell | Bug |
|---------|------------|-----|
| Meaning | Poor coding practice | Programming defect |
| Impact | Affects maintainability | Affects functionality |
| Can application run? | ✅ Yes | ❌ May fail or behave incorrectly |
| Priority | Improve code quality | Fix immediately |

---

# Code Smell

A **Code Smell** is not a bug. It is a sign that the code can be improved to make it more readable, maintainable, or efficient.

## Examples

### Long Method

```java
public void processEmployee() {
    // 300 lines of code
}
```

Better approach:

```text
validateEmployee()
calculateSalary()
saveEmployee()
sendNotification()
```

---

### Duplicate Code

```java
if(user.isAdmin()){
    // logic
}

...

if(user.isAdmin()){
    // same logic
}
```

Move the common logic into a separate method.

---

### Unused Variable

```java
int salary = 50000;
```

If `salary` is never used, it is a code smell.

---

# Bug

A **Bug** is a coding mistake that can cause incorrect results, exceptions, or application failures.

## Examples

### Null Pointer

```java
String name = null;
System.out.println(name.length());
```

Result:

```
NullPointerException
```

---

### Divide by Zero

```java
int result = 10 / 0;
```

Result:

```
ArithmeticException
```

---

### Incorrect String Comparison

```java
if(password == "admin")
```

Correct:

```java
if(password.equals("admin"))
```

---

# SonarQube Analysis

During static code analysis, SonarQube identifies:

- Bugs
- Vulnerabilities
- Code Smells
- Security Hotspots
- Code Duplications

---

# Simple Analogy

### Bug

A car's brakes do not work.

➡️ The car is unsafe to drive.

### Code Smell

The car is messy and poorly organized.

➡️ It still works, but it is difficult to maintain.

---

# Interview Answer

> A **Bug** is a programming defect that may cause incorrect behavior, runtime errors, or application failure. A **Code Smell** is not a functional defect but an indication of poor coding practices that reduce readability, maintainability, or extensibility. SonarQube detects both to help improve software reliability and code quality.
