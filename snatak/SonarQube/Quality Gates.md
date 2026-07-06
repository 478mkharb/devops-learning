# Main Components While Configuring a Quality Gate in SonarQube

A **Quality Gate** is a set of **conditions (thresholds)** that determines whether a project **Passes** or **Fails** after code analysis.

---

# Main Components of a Quality Gate

| Component | Description | Example |
|-----------|-------------|---------|
| **Metric** | The code quality parameter that SonarQube evaluates. | Coverage, Bugs, Vulnerabilities |
| **Operator** | Defines how the metric is compared. | Greater than (>), Less than (<), Equal (=) |
| **Threshold Value** | The acceptable limit for the metric. | Coverage > 80% |
| **Scope** | Determines whether the condition applies to **New Code** or **Overall Code**. | New Code, Overall Code |
| **Quality Gate Result** | Final status after evaluating all conditions. | Pass or Fail |

---

# Common Metrics Used in Quality Gates

| Metric | Purpose | Typical Enterprise Threshold |
|--------|---------|------------------------------|
| **Coverage** | Unit test coverage percentage | > 80% |
| **Duplicated Lines (%)** | Percentage of duplicated code | < 3% |
| **Bugs** | Number of detected bugs | = 0 |
| **Vulnerabilities** | Number of security vulnerabilities | = 0 |
| **Security Rating** | Overall security grade | A |
| **Reliability Rating** | Overall reliability grade | A |
| **Maintainability Rating** | Technical debt and maintainability | A |
| **Security Hotspots Reviewed** | Percentage of reviewed security hotspots | 100% |
| **Code Smells** | Maintainability issues | Organization-specific |
| **Technical Debt** | Estimated effort to fix maintainability issues | Organization-specific |

---

# Example Quality Gate

| Metric | Operator | Threshold |
|--------|----------|-----------|
| Coverage | Greater than | 80% |
| Bugs | Equal | 0 |
| Vulnerabilities | Equal | 0 |
| Duplicated Lines | Less than | 3% |
| Security Rating | Equal | A |
| Reliability Rating | Equal | A |
| Maintainability Rating | Equal | A |

---

# New Code vs Overall Code

| Scope | Description | Recommended |
|-------|-------------|-------------|
| **New Code** | Conditions apply only to newly added or modified code. | ✅ Yes (Best Practice) |
| **Overall Code** | Conditions apply to the entire codebase. | Legacy projects |

---

# Quality Gate Evaluation Example

Suppose the Quality Gate is configured as:

| Metric | Condition |
|--------|-----------|
| Coverage | > 80% |
| Bugs | = 0 |
| Vulnerabilities | = 0 |
| Duplicated Lines | < 3% |

### Analysis Result

| Metric | Actual Value | Status |
|--------|-------------:|--------|
| Coverage | 85% | ✅ Pass |
| Bugs | 0 | ✅ Pass |
| Vulnerabilities | 1 | ❌ Fail |
| Duplicated Lines | 2% | ✅ Pass |

### Final Result

```text
QUALITY GATE

FAILED

Reason:
1 Critical Vulnerability Found
```

---

# Frequently Used Enterprise Quality Gate Conditions

| Condition | Why It Is Used |
|-----------|----------------|
| No New Bugs | Prevents introducing new defects |
| No New Vulnerabilities | Improves application security |
| Coverage on New Code > 80% | Ensures new code is tested |
| Duplicated Lines on New Code < 3% | Prevents code duplication |
| Security Rating = A | Maintains high security standards |
| Reliability Rating = A | Ensures stable and reliable code |
| Maintainability Rating = A | Keeps technical debt under control |

---

# Interview Questions

| Question | Answer |
|----------|--------|
| What is a Quality Gate? | A set of pass/fail conditions that determine whether a project meets predefined quality standards after analysis. |
| What are the main components of a Quality Gate? | Metric, Operator, Threshold Value, Scope (New Code/Overall Code), and Quality Gate Result. |
| What is a metric in a Quality Gate? | A measurable code quality parameter such as Coverage, Bugs, or Vulnerabilities. |
| What is the difference between New Code and Overall Code? | New Code evaluates only recently added or modified code, while Overall Code evaluates the entire codebase. |
| Which metrics are most commonly used in enterprise Quality Gates? | Coverage, Bugs, Vulnerabilities, Duplicated Code, Security Rating, Reliability Rating, and Maintainability Rating. |
| What happens if a Quality Gate fails? | The CI/CD pipeline can be configured to fail, preventing deployment until the issues are resolved. |

---

# One-Line Interview Answer

> **A SonarQube Quality Gate consists of metrics, comparison operators, threshold values, and scope (New Code or Overall Code). After analysis, SonarQube evaluates these conditions and marks the project as Pass or Fail, enabling CI/CD pipelines to enforce code quality standards.**
