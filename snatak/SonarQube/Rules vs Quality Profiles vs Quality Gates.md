# SonarQube: Rules vs Quality Profiles vs Quality Gates

The relationship is simple:

```text
Rules
   │
   ▼
Quality Profile
   │
   ▼
Project Analysis
   │
   ▼
Quality Gate
```

- **Rules** → Individual coding checks.
- **Quality Profile** → Collection of Rules.
- **Quality Gate** → Conditions that determine whether the project passes or fails.

---

# Rules vs Quality Profiles vs Quality Gates

| Feature | Rules | Quality Profile | Quality Gate |
|---------|-------|-----------------|--------------|
| **Definition** | Individual coding rules used to analyze source code. | A collection of rules assigned to a programming language. | A set of pass/fail conditions applied after analysis. |
| **Purpose** | Detect bugs, vulnerabilities, code smells, and maintainability issues. | Decide which rules should be applied during analysis. | Decide whether the analyzed project meets the required quality standards. |
| **Applied To** | Source Code | Programming Language | Entire Project |
| **Configuration Level** | Rule Level | Language Level | Project Level |
| **Examples** | No unused variables, avoid SQL Injection, remove duplicate code. | Java Profile, Python Profile, Go Profile. | Coverage > 80%, No Blocker Bugs, No Critical Vulnerabilities. |
| **Can be Customized?** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Evaluation Time** | During code analysis | Before analysis starts | After analysis completes |
| **Output** | Issues detected | Determines which issues are checked | Pass or Fail |

---

# 1. Rules

## What are Rules?

Rules are **individual coding standards or checks** that SonarQube applies to source code.

Each rule detects one specific problem.

Examples

| Rule | Detects |
|------|----------|
| Unused Variable | Code Smell |
| SQL Injection | Vulnerability |
| Null Pointer Risk | Bug |
| Duplicate Code | Maintainability Issue |
| Hardcoded Password | Security Issue |
| Empty Catch Block | Code Smell |

---

### Example

```java
String password = "admin123";
```

Rule

```
Hardcoded Credentials
```

Result

```
Security Hotspot
```

---

## Interview Answer

> A Rule is an individual code analysis check used by SonarQube to detect bugs, vulnerabilities, code smells, or maintainability issues.

---

# 2. Quality Profile

## What is a Quality Profile?

A Quality Profile is a **collection of Rules** assigned to a specific programming language.

Example

```
Java Quality Profile

↓

Rule 1

Rule 2

Rule 3

Rule 4

Rule 5
```

When SonarQube analyzes Java code,

it applies all the rules inside that Quality Profile.

---

## Example

Java Profile

```
Unused Variables

SQL Injection

Code Duplication

Cyclomatic Complexity

Null Pointer

Memory Leak
```

Python Profile

```
PEP8

Unused Imports

Indentation

Duplicate Code
```

---
# Types of Quality Profiles

| Type | Description |
|------|-------------|
| **Built-in Quality Profile** | Default profile provided by SonarQube (e.g., **Sonar way**). |
| **Custom Quality Profile** | User-created profile by copying or creating a new profile and customizing rules. |

## Interview Answer

> A Quality Profile is a collection of coding rules for a specific programming language that SonarQube applies during static code analysis.

---

# 3. Quality Gate

## What is a Quality Gate?

A Quality Gate is a **set of pass/fail conditions** applied **after** SonarQube completes analysis.

It determines whether the project is ready to proceed in the CI/CD pipeline.

---

## Example

Quality Gate

```
Coverage > 80%

No Blocker Bugs

No Critical Vulnerabilities

Duplicated Code < 3%
```

---

### Analysis Result

```
Coverage

85%

Blocker Bugs

0

Critical Vulnerabilities

0
```

Result

```
PASS
```

---

Another Example

```
Coverage

52%

Critical Vulnerability

2
```

Result

```
FAIL
```

Pipeline stops.

---

## Interview Answer

> A Quality Gate is a set of quality thresholds that determines whether a project passes or fails after SonarQube analysis.

---

# Relationship

```text
Developer Code

        │

        ▼

SonarQube Rules

        │

        ▼

Quality Profile

(Which Rules?)

        │

        ▼

Code Analysis

        │

        ▼

Quality Gate

(Pass / Fail)

        │

        ▼

CI/CD Decision
```

---

# Real Example

Suppose you have

Employee API (Go)

Quality Profile

```
Go Profile

↓

Unused Variables

↓

Error Handling

↓

Code Duplication

↓

Cyclomatic Complexity
```

Analysis Result

```
Coverage = 82%

Critical Bugs = 0

Vulnerabilities = 0
```

Quality Gate

```
Coverage > 80%

↓

PASS
```

Deployment continues.

---

# Quick Comparison

| Rules | Quality Profile | Quality Gate |
|--------|-----------------|--------------|
| Individual Checks | Collection of Rules | Pass/Fail Conditions |
| Detect Problems | Decide What to Check | Decide Whether to Release |
| During Analysis | Before Analysis | After Analysis |
| Code Level | Language Level | Project Level |

---

# Frequently Asked Interview Questions

| Question | Answer |
|----------|--------|
| What is a Rule in SonarQube? | An individual coding check that detects bugs, vulnerabilities, or code smells. |
| What is a Quality Profile? | A collection of rules assigned to a programming language. |
| What is a Quality Gate? | A set of quality conditions that determines whether a project passes or fails after analysis. |
| Can Rules be customized? | Yes. |
| Can Quality Profiles be customized? | Yes. |
| Can Quality Gates be customized? | Yes. |
| Which component decides PASS or FAIL? | Quality Gate. |
| Which component decides what to analyze? | Quality Profile. |
| Which component actually detects issues? | Rules. |

---

# One-Line Interview Answers

| Component | One-Line Answer |
|-----------|-----------------|
| **Rule** | An individual coding check that detects bugs, vulnerabilities, or code smells. |
| **Quality Profile** | A collection of rules assigned to a programming language for code analysis. |
| **Quality Gate** | A set of pass/fail conditions that determines whether a project meets the required quality standards after analysis. |
