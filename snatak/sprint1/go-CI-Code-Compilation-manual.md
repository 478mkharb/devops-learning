# <h1 align="center">POC - GO Lang CI | Code Compilation </h1>

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
    <td align="center">17/05/2026</td>
    <td align="center">1.0</td>
    <td align="center">Mukesh Kharb</td>
    <td align="center">17/05/2026</td>
    <td align="center">Team</td>
    <td align="center">Mohit Kumar</td>
    <td align="center">Faisal Khan</td>
    <td align="center">Mahesh Kumar</td>
  </tr>

  <tr>
    <td align="center">Mukesh Kharb</td>
    <td align="center">19/05/2026</td>
    <td align="center">1.1</td>
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

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [Install GO Language](#3-install-go-language)
4. [Clone the Repository](#4-clone-the-repository)
5. [Verify GO Modules](#5-verify-go-modules)
6. [Download Dependencies](#6-download-dependencies)
7. [Compile the Application and Verify Binary Generation](#7-compile-the-application-and-verify-binary-generation)
8. [Run the Application](#8-run-the-application)
9. [Validate Application Health](#9-validate-application-health)
10. [Common Code Compilation Errors](#10-common-code-compilation-errors)
11. [Conclusion](#11-conclusion)
12. [Contact Information](#12-contact-information)
13. [References](#13-references)

---

<a id="1-introduction"></a>

## 1. Introduction

This POC demonstrates manual GO code compilation for GO-based applications in OT-Microservices. The process validates dependency resolution, package imports, application build generation, and runtime execution before deployment.

The POC ensures:

* Successful GO application compilation
* Dependency validation using GO Modules
* Binary generation verification
* Runtime execution testing
* Manual build validation workflow

---

<a id="2-pre-requisites"></a>

## 2. Pre-requisites

| Component | Version |
|---|---|
| Ubuntu Server | 22.04 LTS | 
| GO Language | 1.22.5 | 

---

<a id="3-install-go-language"></a>

## 3. Install GO Language

Download Latest GO:

```bash
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
```
Extract GO
```bash
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
```
Configure Path

```bash
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```
Verify GO installation:

```bash
go version
```

<details>
<summary>📸 <strong>Click to view Screenshot - Install GO Language</strong></summary>
<img width="1420" height="735" alt="image" src="https://github.com/user-attachments/assets/809817bb-0f77-47f3-8d8e-9941b510efef" />


</details>

---

<a id="4-clone-the-repository"></a>

## 4. Clone the Repository

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
<details>
<summary>📸 <strong>Click to view Screenshot - Clone the Repository</strong></summary>
<img width="1420" height="735" alt="image" src="https://github.com/user-attachments/assets/d0c9436d-92d3-46f0-89a5-da7c3f1c2ce0" />

</details>

---

<a id="5-verify-go-modules"></a>

## 5. Verify GO Modules

Verify GO module configuration:

```bash
cat go.mod
```
Verify environment:

```bash
go env
```

---

<a id="6-download-dependencies"></a>

## 6. Download Dependencies

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
<details>
<summary>📸 <strong>Click to view Screenshot - Download Dependencies</strong></summary>
<img width="1442" height="907" alt="image" src="https://github.com/user-attachments/assets/a0a77496-8728-482a-9f44-1f719d3dba2c" />

</details>

---

<a id="7-compile-the-application-and-verify-binary-generation"></a>

## 7. Compile the Application and Verify Binary Generation

Compile application:

```bash
go build -o employee-api
```
If compilation succeeds, no error output appears.

```bash
ls -l
```

<details>
<summary>📸 <strong>Click to view Screenshot - Compile the Application and Verify Binary Generation</strong></summary>
<img width="1387" height="713" alt="image" src="https://github.com/user-attachments/assets/292692d9-937b-439f-a907-dcfecac8cc92" />

</details>

---

<a id="8-run-the-application"></a>

## 8. Run the Application

Start the application in background (release mode):

```bash
export GIN_MODE=release
nohup ./employee-api > app.log 2>&1 &
```

Verify running process:

```bash
ps aux | grep employee-api
```

Verify listening ports:

```bash
ss -tulpn | grep 8080
```
<details>
<summary>📸 <strong>Click to view Screenshot - Run the Application</strong></summary>
<img width="1397" height="292" alt="image" src="https://github.com/user-attachments/assets/39f1581d-df48-46f0-860e-337137352ff9" />

</details>

---

<a id="9-validate-application-health"></a>

## 9. Validate Application Health

Run health check:

```bash
curl http://localhost:8080/api/v1/employee/health
```
<details>
<summary>📸 <strong>Click to view Screenshot - Validate Application Health</strong></summary>
<img width="1418" height="324" alt="image" src="https://github.com/user-attachments/assets/cdbbf7e7-4f93-401f-b422-b081ee09a91f" />

</details>

---

<a id="10-common-code-compilation-errors"></a>

## 10. Common Code Compilation Errors

| Error                                                                  | Cause                                                    | Solution                                         |
| ---------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------ |
| `missing go.sum entry`                                                 | Missing or inconsistent dependency checksums             | Run `go mod tidy`                                |
| `package not found`                                                    | Required GO module or package is missing                 | Verify import path and download dependencies     |
| `undefined: <function/variable>`                                       | Undefined function, variable, or package reference       | Verify code syntax and package imports           |
| `permission denied`                                                    | Binary does not have execute permission                  | Run `chmod +x <binary-name>`                     |
| `listen tcp :8080: bind: address already in use`                       | Another application is already using the port            | Stop existing process or change application port |
| `go.mod file indicates go 1.20, but maximum version supported is 1.18` | Installed GO version is outdated                         | Upgrade GO to required version                   |
| `cannot find module providing package`                                 | Dependency module not downloaded                         | Run `go get <module-name>` or `go mod tidy`      |
| `build failed`                                                         | Syntax error, dependency issue, or configuration problem | Review build logs and fix reported errors        |


---

<a id="11-conclusion"></a>

## 11. Conclusion

Manual GO code compilation ensures application stability before deployment. Using `go mod tidy` and `go build` validates dependencies, package imports, and executable generation. This process helps developers detect build issues early and maintain consistent application delivery workflows.

---

<a id="12-contact-information"></a>

## 12. Contact Information

| Name         | ✉️ Contact                                                                        |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

<a id="13-references"></a>

## 13. References

| S.No | Description               | Click to View |
| ---- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1 | GO Official Documentation | [![GO Docs](https://img.shields.io/badge/GO-DOCUMENTATION-2F2F2F?style=flat-square&logo=go&logoColor=white)](https://go.dev/doc/) |
| 2 | GO Modules Documentation | [![GO Modules](https://img.shields.io/badge/GO-MODULES-3A3A3A?style=flat-square&logo=go&logoColor=white)](https://go.dev/ref/mod) |
| 3 | GitHub Documentation | [![GitHub](https://img.shields.io/badge/GITHUB-DOCUMENTATION-1F1F1F?style=flat-square&logo=github&logoColor=white)](https://docs.github.com/) |
| 4 | GNU Make Documentation | [![MAKE](https://img.shields.io/badge/MAKE-DOCUMENTATION-404040?style=flat-square&logo=gnu&logoColor=white)](https://www.gnu.org/software/make/manual/make.html) |
