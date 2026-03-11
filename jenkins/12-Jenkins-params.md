# Jenkins UI Parameters (Parameterized Builds)

## 1. What are Parameters in Jenkins

Jenkins parameters allow users to pass values to a job at runtime when triggering a build.

When **"This project is parameterized"** is enabled in Jenkins job configuration, Jenkins UI provides different parameter types that can be used inside pipelines or freestyle jobs.

These parameters make pipelines flexible and reusable.

Example use cases:

* Select deployment environment (dev/stage/prod)
* Choose application version
* Enable or disable specific steps

---

# 2. Common Jenkins Parameter Types

## 2.1 String Parameter

Allows users to enter a text value.

Example configuration:

Parameter Name: VERSION
Default Value: 1.0

Example usage in pipeline:

```
echo "Deploying version ${params.VERSION}"
```

Example UI Input:

```
VERSION = 2.1
```

---

## 2.2 Boolean Parameter

Provides a checkbox (true/false).

Example:

Parameter Name: RUN_TESTS
Default: true

Pipeline usage:

```
if(params.RUN_TESTS){
   echo "Running tests"
}
```

---

## 2.3 Choice Parameter

Allows selecting one option from a dropdown list.

Example configuration:

Parameter Name: ENVIRONMENT
Choices:

```
dev
stage
prod
```

Pipeline usage:

```
echo "Deploying to ${params.ENVIRONMENT}"
```

---

## 2.4 Text Parameter

Used for multi-line input values.

Example:

Parameter Name: RELEASE_NOTES

Example input:

```
Bug fixes
Performance improvements
```

---

## 2.5 Password Parameter

Used to pass secret values.

Example:

Parameter Name: DB_PASSWORD

Value is hidden in Jenkins UI.

---

## 2.6 File Parameter

Allows uploading a file during build execution.

Example use case:

Upload configuration file.

Example:

```
config.yaml
```

---

## 2.7 Run Parameter

Used to select another Jenkins job build.

Example:

```
Select build from job: build-app
```

This is useful when pipelines depend on artifacts from previous builds.

---

# 3. Example Parameterized Pipeline

```
pipeline {
  agent any

  parameters {
    string(name: 'VERSION', defaultValue: '1.0')
    choice(name: 'ENV', choices: ['dev','stage','prod'])
    booleanParam(name: 'RUN_TESTS', defaultValue: true)
  }

  stages {
    stage('Build') {
      steps {
        echo "Building version ${params.VERSION}"
      }
    }

    stage('Test') {
      when {
        expression { params.RUN_TESTS }
      }
      steps {
        echo "Running tests"
      }
    }

    stage('Deploy') {
      steps {
        echo "Deploying to ${params.ENV}"
      }
    }
  }
}
```

---

# 4. How to Enable Parameters in Jenkins UI

Steps:

1. Open Jenkins Job
2. Click **Configure**
3. Enable **This project is parameterized**
4. Add parameter type
5. Save job

When you click **Build with Parameters**, Jenkins shows the parameter input form.

---

# 5. Real DevOps Example

Deployment pipeline parameters:

```
ENVIRONMENT = dev / stage / prod
VERSION = docker image tag
RUN_DB_MIGRATION = true/false
```

This allows the same pipeline to deploy to multiple environments.

---

# 6. Interview Explanation

Jenkins parameters are inputs provided at build time that allow users to control pipeline behavior dynamically. Jenkins supports different parameter types such as string, choice, boolean, text, password, file, and run parameters. These parameters make Jenkins jobs reusable and configurable for different environments or versions.
