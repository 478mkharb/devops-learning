# 🔥 Jenkins Declarative vs Scripted Pipeline — Detailed Guide (with Explanations)

---

## 1. Checkout SCM

### Explanation

This defines how source code is fetched from the repository.

* Declarative pipelines automatically perform a checkout of the configured SCM.
* Scripted pipelines require explicit checkout, otherwise workspace will be empty.

### Declarative

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'ls'
            }
        }
    }
}
```

### Scripted

```groovy
node {
    stage('Checkout') {
        checkout scm
    }
}
```

---

## 2. Restart from Stage

### Explanation

Allows restarting pipeline execution from a failed stage instead of running entire pipeline again.

* Declarative supports this via Jenkins UI.
* Scripted does not support it because flow is not strictly defined.

---

## 3. Starting Block

### Explanation

Defines how pipeline execution begins and allocates executor.

* Declarative uses `pipeline {}` abstraction.
* Scripted directly uses `node {}` to allocate executor.

### Declarative

```groovy
pipeline {
    agent any
}
```

### Scripted

```groovy
node {
}
```

---

## 4. Pipeline Structure

### Explanation

Refers to how strictly the pipeline is organized.

* Declarative enforces structure (pipeline → stages → steps).
* Scripted allows free-form logic.

### Declarative

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Hello'
            }
        }
    }
}
```

### Scripted

```groovy
node {
    stage('Build') {
        echo 'Hello'
    }
}
```

---

## 5. Groovy vs DSL

### Explanation

Defines how much programming flexibility is available.

* Declarative is DSL-based → controlled syntax
* Scripted is full Groovy → complete flexibility

### Declarative

```groovy
steps {
    script {
        def x = 10
        echo "Value: ${x}"
    }
}
```

### Scripted

```groovy
node {
    def x = 10
    echo "Value: ${x}"
}
```

---

## 6. Error Handling

### Explanation

How failures are handled and post-actions are executed.

* Declarative uses `post` blocks
* Scripted uses Groovy try-catch

### Declarative

```groovy
post {
    failure {
        echo 'Failed'
    }
}
```

### Scripted

```groovy
node {
    try {
        sh 'false'
    } catch (e) {
        echo 'Failed'
    }
}
```

---

## 7. Parallel Stages

### Explanation

Used to execute multiple tasks simultaneously.

* Declarative provides structured syntax
* Scripted uses Groovy map

### Declarative

```groovy
parallel {
    stage('Build') {
        steps { echo 'Build' }
    }
    stage('Test') {
        steps { echo 'Test' }
    }
}
```

### Scripted

```groovy
node {
    parallel(
        build: { echo 'Build' },
        test: { echo 'Test' }
    )
}
```

---

## 8. Syntax Validation

### Explanation

Syntax validation defines **when Jenkins checks your pipeline code for correctness** — either before execution starts or during execution.

This is important because it directly impacts:

* Build failures
* Resource usage (agents, time)
* Debugging complexity

---

### Declarative Pipeline (Pre-validation) ✅

Jenkins validates the entire pipeline before execution begins.

#### What gets validated:

* Proper structure (`pipeline → stages → steps`)
* Required blocks present
* Syntax correctness
* Misplaced directives

#### Example (Error caught BEFORE execution)

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            echo 'Hello'   // ❌ Missing steps block
        }
    }
}
```

👉 Result:

* Pipeline **does NOT start**
* Jenkins throws error immediately

---

### Scripted Pipeline (Runtime validation) ❌

Jenkins executes the pipeline line-by-line and validates during execution.

#### Behavior:

* Starts execution immediately
* Errors appear only when problematic line is reached

#### Example

```groovy
node {
    stage('Build') {
        echo 'Start Build'
        sh 'invalid-command'   // ❌ error here
    }

    stage('Deploy') {
        echo 'Deploying'
    }
}
```

#### Execution Flow:

1. "Start Build" runs ✅
2. Error occurs at `invalid-command` ❌
3. Pipeline stops midway
4. Previous steps already executed

---

### 🔥 Key Difference

* Declarative → **Fail Fast (before execution)**
* Scripted → **Fail Late (during execution)**

---

### 🔥 Real DevOps Impact

| Scenario          | Declarative     | Scripted                       |
| ----------------- | --------------- | ------------------------------ |
| Wrong syntax      | Fails instantly | Fails after partial run        |
| Resource usage    | Saved           | Wasted (agent already running) |
| Debugging         | Easier          | Harder                         |
| CI/CD reliability | High            | Medium                         |

---

## 9. Parameters Definition Usage

### Explanation

Defines how user inputs are passed to pipeline.

* Declarative uses clean `parameters {}` block
* Scripted uses `properties()` which overwrites job config

### Declarative

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'ENV', defaultValue: 'dev')
    }

    stages {
        stage('Print') {
            steps {
                echo "ENV is ${params.ENV}"
            }
        }
    }
}
```

### Scripted

```groovy
node {
    properties([
        parameters([
            string(name: 'ENV', defaultValue: 'dev')
        ])
    ])

    stage('Print') {
        echo "ENV is ${params.ENV}"
    }
}
```

---

# 🔥 Summary Table

| Feature         | Declarative   | Scripted     |
| --------------- | ------------- | ------------ |
| Checkout        | Auto          | Manual       |
| Restart Stage   | Yes           | No           |
| Start Block     | pipeline      | node         |
| Structure       | Fixed         | Flexible     |
| Language        | DSL           | Groovy       |
| Error Handling  | post          | try-catch    |
| Parallel        | Structured    | Groovy map   |
| Validation      | Pre-run       | Runtime      |
| Parameters      | parameters {} | properties() |
| Maintainability | High          | Medium       |

---

# 🚀 Final Summary

* Declarative → Easy, structured, safer
* Scripted → Flexible, powerful, complex

---
