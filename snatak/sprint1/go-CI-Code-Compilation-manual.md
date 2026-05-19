# <h1 align="center">POC - GO Lang Manual Build | Code Compilation </h1>

<div align="center">
<img width="150" alt="GoLang" src="https://go.dev/blog/go-brand/Go-Logo/PNG/Go-Logo_Blue.png" />
</div>

<br/>

---

<div align="center">

<table>
  <tr>
    <th align="center">Author</th>
    <th align="center">Created On</th>
    <th align="center">Version</th>
    <th align="center">Last Updated By</th>
    <th align="center">Last Edited On</th>
    <th align="center">Pre Reviewer</th>
    <th align="center">L0 Reviewer</th>
    <th align="center">L1 Reviewer</th>
    <th align="center">L2 Reviewer</th>
  </tr>

  <tr>
    <td align="center">Mukesh Kharb</td>
    <td align="center">19/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">19/05/2026</td>
    <td align="center">Team</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>
</table>

</div>

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [Install GO Language](#3-install-go-language)
4. [Clone the Repository](#4-clone-the-repository)
5. [Verify GO Modules](#5-verify-go-modules)
6. [Download Dependencies](#6-download-dependencies)
7. [Compile the Application](#7-compile-the-application)
8. [Verify Binary Generation](#8-verify-binary-generation)
9. [Run the Application](#9-run-the-application)
10. [Validate Application Health](#10-validate-application-health)
11. [Common Build Validation Commands](#11-common-build-validation-commands)
12. [Common Compilation Errors](#12-common-compilation-errors)
13. [Conclusion](#13-conclusion)
14. [Contact Information](#14-contact-information)
15. [References](#15-references)

---

<a id="1-introduction"></a>

# 1. Introduction

This POC demonstrates manual GO code compilation for GO-based applications in OT-Microservices. The process validates dependency resolution, package imports, application build generation, and runtime execution before deployment.

The POC ensures:

* Successful GO application compilation
* Dependency validation using GO Modules
* Binary generation verification
* Runtime execution testing
* Manual build validation workflow

---

<a id="2-pre-requisites"></a>

# 2. Pre-requisites

| Requirement           | Details                         |
| --------------------- | ------------------------------- |
| Operating System      | Ubuntu 22.04 or later           |
| Access                | Sudo Privileges                 |
| RAM                   | Minimum 2 GB                    |
| Disk Space            | Minimum 5 GB                    |
| Internet Connectivity | Required                        |
| GitHub Access         | Required for repository cloning |

---

<a id="3-install-go-language"></a>

# 3. Install GO Language

Update system packages:

```bash
sudo apt update
```

Install GO:

```bash
sudo apt install golang-go -y
```

Verify GO installation:

```bash
go version
```

Expected Output:

```text
go version go1.22.x linux/amd64
```

---

<a id="4-clone-the-repository"></a>

# 4. Clone the Repository

Clone the GO application repository:

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api.git
```

Move into project directory:

```bash
cd employee-api
```

Verify files:

```bash
ls
```

Expected Files:

```text
go.mod
go.sum
main.go
```

---

<a id="5-verify-go-modules"></a>

# 5. Verify GO Modules

Verify GO module configuration:

```bash
cat go.mod
```

Example Output:

```go
module employee-api

go 1.22
```

Verify environment:

```bash
go env
```

---

<a id="6-download-dependencies"></a>

# 6. Download Dependencies

Run dependency validation:

```bash
go mod tidy
```

This command performs:

* Dependency download
* Module verification
* Unused dependency cleanup
* `go.sum` validation

Verify modules:

```bash
go list -m all
```

---

<a id="7-compile-the-application"></a>

# 7. Compile the Application

Compile application:

```bash
go build
```

Or generate custom binary:

```bash
go build -o employee-api
```

If compilation succeeds, no error output appears.

---

<a id="8-verify-binary-generation"></a>

# 8. Verify Binary Generation

Verify generated binary:

```bash
ls -l
```

Expected Output:

```text
employee-api
```

Verify binary type:

```bash
file employee-api
```

Expected Output:

```text
ELF 64-bit executable
```

---

<a id="9-run-the-application"></a>

# 9. Run the Application

Start the application:

```bash
./employee-api
```

Or run in background:

```bash
nohup ./employee-api > app.log 2>&1 &
```

Verify running process:

```bash
ps aux | grep employee-api
```

Verify listening ports:

```bash
ss -tulpn
```

---

<a id="10-validate-application-health"></a>

# 10. Validate Application Health

Run health check:

```bash
curl http://localhost:8080/api/v1/employee/health
```

Expected Response:

```json
{"message":"Employee API is up and running"}
```

---

<a id="11-common-build-validation-commands"></a>

# 11. Common Build Validation Commands

| Command          | Purpose                  |
| ---------------- | ------------------------ |
| `go version`     | Verify GO installation   |
| `go env`         | Verify GO environment    |
| `go mod tidy`    | Validate dependencies    |
| `go build`       | Compile application      |
| `go test ./...`  | Run unit tests           |
| `go list -m all` | Verify installed modules |
| `file <binary>`  | Verify executable format |

---

<a id="12-common-compilation-errors"></a>

# 12. Common Compilation Errors

| Error                  | Cause                  | Solution                |
| ---------------------- | ---------------------- | ----------------------- |
| `missing go.sum entry` | Dependency mismatch    | Run `go mod tidy`       |
| `package not found`    | Missing dependency     | Download modules        |
| `undefined function`   | Import issue           | Verify package imports  |
| `permission denied`    | Binary not executable  | Run `chmod +x <binary>` |
| `build failed`         | Syntax or module issue | Review build logs       |

---

<a id="13-conclusion"></a>

# 13. Conclusion

Manual GO code compilation ensures application stability before deployment. Using `go mod tidy` and `go build` validates dependencies, package imports, and executable generation. This process helps developers detect build issues early and maintain consistent application delivery workflows.

---

<a id="14-contact-information"></a>

# 14. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="15-references"></a>

# 15. References

| S.No | Description               | Click to View                                                                                            |
| ---- | ------------------------- | -------------------------------------------------------------------------------------------------------- |
| 1    | GO Official Documentation | [https://go.dev/doc/](https://go.dev/doc/)                                                               |
| 2    | GO Modules Documentation  | [https://go.dev/ref/mod](https://go.dev/ref/mod)                                                         |
| 3    | GitHub Documentation      | [https://docs.github.com/](https://docs.github.com/)                                                     |
| 4    | GNU Make Documentation    | [https://www.gnu.org/software/make/manual/make.html](https://www.gnu.org/software/make/manual/make.html) |
