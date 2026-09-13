# Linux with CI/CD Tools — DevOps Interview Handbook

> **Focus:** Linux from a DevOps and CI/CD engineering perspective.  
> **Examples:** Ubuntu-based Jenkins controllers/agents, EC2 build hosts, systemd services, Ansible, Terraform, Packer, Maven, Java, Python, Node.js, Go, Git, and NGINX.

---

## 1. Why Linux Matters in CI/CD

Most CI/CD tools execute commands on Linux hosts:

```text
Git repository
     |
     v
CI controller
     |
     v
Build agent
     |
     +--> Git checkout
     +--> Dependency download
     +--> Compile/build
     +--> Unit tests
     +--> Security scans
     +--> Package/artifact creation
     +--> Deployment
     |
     v
Target server / EC2 / Kubernetes / artifact repository
```

Linux is responsible for process execution, file permissions, workspaces, environment variables, networking, tool installation, caching, artifact storage, service management, resource isolation, logs, and security boundaries.

A pipeline can fail even when application code is correct because the Linux environment is incorrect.

---

## 2. CI Controller vs CI Agent

### CI controller

The controller coordinates jobs and schedules work.

Responsibilities:

- Store pipeline configuration
- Schedule builds
- Manage credentials and plugins
- Display build results
- Assign work to agents
- Maintain build metadata

### CI agent

The agent executes build commands.

Responsibilities:

- Clone source code
- Run Maven, Gradle, npm, Python, or Go
- Execute tests
- Run scanners
- Build packages
- Upload artifacts
- Deploy applications

A common design is:

```text
Jenkins Controller
       |
       +--> Java Build Agent
       +--> Node.js Build Agent
       +--> Security Scan Agent
       +--> Infrastructure Agent
```

Benefits:

- Reduced controller load
- Better security isolation
- Different tool versions per agent
- Easier scaling
- Easier troubleshooting
- Less dependency conflict

---

## 3. Jenkins Linux Service User

Inspect Jenkins:

```bash
ps -ef | grep -i jenkins
systemctl status jenkins
systemctl cat jenkins
systemctl show jenkins -p User -p Group
id jenkins
getent passwd jenkins
```

A CI process should not normally run as root. A dedicated account limits file access, deployment permissions, credential exposure, and damage from malicious build scripts.

Bad practice:

```bash
sudo chmod -R 777 /var/lib/jenkins
```

Better:

```bash
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/project
sudo chmod -R u+rwX,go-rwx /var/lib/jenkins/workspace/project
```

Use the narrowest permissions required.

---

## 4. Jenkins Workspace

A workspace is where a job checks out and builds code.

Typical locations:

```text
/var/lib/jenkins/workspace/<job-name>
/home/jenkins/agent/workspace/<job-name>
```

Inspect it:

```bash
pwd
ls -lah
du -sh "$WORKSPACE"
df -h "$WORKSPACE"
ls -ld "$WORKSPACE"
stat "$WORKSPACE"
```

Common workspace problems:

- Previous build files remain
- Wrong owner after a manual `sudo` command
- Disk becomes full
- Old artifacts mix with new artifacts
- Multiple jobs write to one directory
- A previous process remains running
- Git checkout becomes corrupted

Safer cleanup:

```bash
if [ -n "${WORKSPACE:-}" ] && [ -d "$WORKSPACE" ]; then
    find "$WORKSPACE" -mindepth 1 -maxdepth 1 -exec rm -rf -- {} +
fi
```

Prefer the CI system's cleanup mechanism. Never blindly execute `rm -rf` against an unvalidated path.

---

## 5. Linux Executors and Parallel Builds

An executor is a slot that can run one build at a time.

Too many executors can cause:

- CPU saturation
- Memory pressure
- OOM kills
- Disk I/O contention
- Network saturation
- Dependency repository throttling
- Port conflicts
- Test instability

Inspect resources:

```bash
nproc
lscpu
uptime
top
free -h
vmstat 1 5
iostat -xz 1 5
cat /proc/loadavg
```

Choose executor count based on CPU, memory, build type, integration tests, disk speed, and network requirements.

---

## 6. Java Version Problems

Jenkins, Maven plugins, and application source code may require different Java versions.

Common errors:

```text
release version 8 not supported
Unsupported class file major version
invalid target release: 8
```

Inspect Java:

```bash
java -version
javac -version
which java
readlink -f "$(which java)"
update-java-alternatives --list
ls -lah /usr/lib/jvm
```

Example:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"

java -version
mvn -version
```

The Java runtime used by Jenkins is not necessarily the Java version required to compile the application.

A valid design may be:

```text
Jenkins runtime: Java 17
Application build: Java 8 toolchain
```

Use Maven toolchains, explicit compiler configuration, or separate agents.

Diagnostics:

```bash
mvn -version
mvn help:effective-pom
mvn -X test
```

---

## 7. Maven Builds

Typical lifecycle:

```text
validate -> compile -> test -> package -> verify -> install -> deploy
```

Commands:

```bash
mvn clean test
mvn clean package
mvn clean verify
mvn -DskipTests package
mvn -U clean verify
mvn -X clean verify
```

Important flags:

| Flag | Meaning |
|---|---|
| `-e` | Show execution error details |
| `-X` | Debug logging |
| `-U` | Force dependency update checks |
| `-DskipTests` | Skip test execution but usually compile tests |
| `-Dmaven.test.skip=true` | Skip test compilation and execution |

Maven cache:

```text
~/.m2/repository
/var/lib/jenkins/.m2/repository
```

Inspect:

```bash
du -sh ~/.m2
find ~/.m2/repository -type f | head
```

If one dependency is corrupted, remove only its directory and retry:

```bash
rm -rf ~/.m2/repository/group/name
mvn -U clean verify
```

---

## 8. JaCoCo, Checkstyle, PMD and Security Scans

A typical Java pipeline includes:

```text
Compile
  |
  +--> Unit tests
  +--> JaCoCo coverage
  +--> Checkstyle
  +--> PMD
  +--> Dependency scan
  +--> Secret scan
  +--> SonarQube analysis
```

Typical failures:

| Tool | Common failure |
|---|---|
| JaCoCo | Agent/plugin mismatch or missing report |
| Checkstyle | Coding standard violation |
| PMD | Static analysis violation |
| GitLeaks | Secret detected |
| SonarQube | Quality gate failure or server unreachable |
| Dependency scan | Vulnerable dependency |
| Surefire | Unit test failure |
| Maven Compiler | Wrong Java version |

Inspect build output:

```bash
find target -maxdepth 3 -type f | sort
find . -iname '*jacoco*'
find . -iname '*surefire*'
ls -lah target
```

Test write access as Jenkins:

```bash
sudo -u jenkins touch target/test-file
```

Do not make the whole filesystem writable to solve a local permission problem.

---

## 9. Node.js and Frontend Builds

Commands:

```bash
node --version
npm --version
npm ci
npm test
npm run build
```

Prefer `npm ci` in CI because it uses the lock file and performs a clean dependency installation.

Example:

```bash
npm ci
npm run lint
npm test -- --ci
npm run build
```

Diagnostics:

```bash
node -v
which node
npm config get cache
du -sh node_modules
du -sh ~/.npm
```

Avoid uncontrolled global installation with `sudo npm install -g`.

Common frontend artifacts:

```text
dist/
build/
```

Validate:

```bash
test -d dist
find dist -maxdepth 2 -type f | head
```

---

## 10. Python Builds

Check Python:

```bash
python3 --version
which python3
```

Use a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
pytest
```

Verify interpreter and pip alignment:

```bash
python -c 'import sys; print(sys.executable)'
python -m pip --version
```

Virtual environments prevent conflicts between system Python and application dependencies.

For `externally-managed-environment`, use a virtual environment instead of installing packages globally.

---

## 11. Go Builds

Commands:

```bash
go version
go env
go mod download
go mod verify
go test ./...
go build ./...
```

Example:

```bash
go build -o bin/employee-api ./cmd/employee-api
file bin/employee-api
ls -lh bin/employee-api
```

Inspect caches:

```bash
go env GOPATH
go env GOMODCACHE
go env GOCACHE
```

Check binary dependencies:

```bash
ldd bin/employee-api
```

For a possible static build:

```bash
CGO_ENABLED=0 go build -o employee-api .
```

Validate target architecture:

```bash
uname -m
go env GOOS GOARCH
```

---

## 12. Git in CI

Typical flow:

```bash
git clone <repository>
cd repository
git checkout <branch>
git rev-parse HEAD
```

Diagnostics:

```bash
git status --short
git branch --show-current
git log -1 --oneline
git remote -v
```

CI systems may check out a commit directly, producing a detached HEAD. That is often normal.

Never hardcode credentials:

```bash
git clone https://user:password@example.com/repo.git
```

Use Jenkins credentials, SSH agents, credential bindings, short-lived tokens, or cloud identity.

Never print secrets through:

```bash
env
set
echo "$PASSWORD"
```

---

## 13. Shell Scripting in Pipelines

Use strict mode:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

Meaning:

- `-e`: exit on command failure
- `-u`: fail on unset variables
- `pipefail`: fail if an earlier pipeline command fails

Example:

```bash
#!/usr/bin/env bash
set -euo pipefail

: "${APP_NAME:?APP_NAME is required}"
: "${ARTIFACT:?ARTIFACT is required}"

test -f "$ARTIFACT"
sha256sum "$ARTIFACT"
echo "Deploying ${APP_NAME}"
```

Common exit codes:

| Code | Meaning |
|---:|---|
| `0` | Success |
| `1` | Generic failure |
| `2` | Shell misuse |
| `126` | Found but not executable |
| `127` | Command not found |
| `128+n` | Terminated by signal `n` |

Avoid hiding failures:

```bash
some_command || true
```

Use it only when failure is intentionally non-fatal.

---

## 14. Traps, Cleanup and Timeouts

Builds can leave temporary files, test servers, lock files, or ports occupied.

Example:

```bash
#!/usr/bin/env bash
set -euo pipefail

tmp_dir="$(mktemp -d)"

cleanup() {
    rm -rf "$tmp_dir"
}

trap cleanup EXIT INT TERM
```

Timeout:

```bash
timeout 10m ./integration-tests.sh
```

A timeout commonly returns exit code `124`.

Background process cleanup:

```bash
./start-test-server.sh &
server_pid=$!

cleanup() {
    kill "$server_pid" 2>/dev/null || true
}

trap cleanup EXIT
wait "$server_pid"
```

---

## 15. Artifact Handling

Artifacts include:

- JAR/WAR
- ZIP/TAR.GZ
- Python wheel
- Go binary
- React `dist/`
- RPM/DEB package
- Terraform plan

Validate:

```bash
test -s target/app.jar
file target/app.jar
sha256sum target/app.jar
```

Archive:

```bash
tar -czf release.tar.gz dist/
tar -tzf release.tar.gz | head
```

Prefer immutable artifact names:

```text
employee-api-1.4.2-commit-a1b2c3d.jar
```

Immutable artifacts improve traceability and rollback. Avoid overwriting a generic `latest.jar`.

---

## 16. Deployment to systemd

Common flow:

```text
Build
  -> Upload artifact
  -> Validate artifact
  -> Install with correct owner/mode
  -> Restart service
  -> Inspect status/logs
  -> Health check
```

Example:

```bash
sudo install -o appuser -g appuser -m 0750     employee-api /opt/employee-api/employee-api

sudo systemctl restart employee-api
sudo systemctl is-active --quiet employee-api
curl --fail http://127.0.0.1:8080/health
```

Diagnostics:

```bash
sudo systemctl status employee-api --no-pager
sudo journalctl -u employee-api -n 100 --no-pager
ss -lntp
```

Use a dedicated service account:

```ini
[Service]
User=employee
Group=employee
ExecStart=/opt/employee-api/employee-api
Restart=on-failure
```

Do not run application services as root without a justified requirement.

---

## 17. NGINX Deployment

Typical frontend flow:

```text
React build
   |
   v
dist/
   |
   v
/var/www/otms-frontend/
   |
   v
NGINX
```

Example:

```bash
sudo rsync -a --delete dist/ /var/www/otms-frontend/
sudo nginx -t
sudo systemctl reload nginx
```

Always validate before reload:

```bash
sudo nginx -t
```

Verify:

```bash
curl -I http://127.0.0.1/
sudo systemctl status nginx --no-pager
sudo journalctl -u nginx -n 50 --no-pager
```

Use `--delete` only when you understand which destination files will be removed.

---

## 18. Ansible from CI

Commands:

```bash
ansible --version
ansible-inventory -i inventory/hosts.yml --graph
ansible all -i inventory/hosts.yml -m ping
ansible-playbook -i inventory/hosts.yml deploy.yml
```

Best practices:

- Use a dedicated automation account
- Store vault passwords securely
- Use SSH keys or SSM-compatible mechanisms
- Keep inventories environment-specific
- Use `--check` where practical
- Pin collection versions
- Validate YAML
- Avoid exposing secrets with `--diff`

Example:

```bash
ansible-playbook   -i inventory/dev.yml   deploy.yml   --limit notification   --extra-vars "release_version=${BUILD_NUMBER}"
```

A common error is:

```text
Missing sudo password
```

Use controlled `become` configuration, limited `NOPASSWD` rules, or a secure secret mechanism. Never commit plaintext passwords.

---

## 19. Terraform and Packer from CI

Terraform sequence:

```bash
terraform fmt -check
terraform init
terraform validate
terraform plan -out=tfplan
terraform apply tfplan
```

Destroy workflow:

```bash
terraform plan -destroy -out=destroy.tfplan
terraform apply destroy.tfplan
```

Packer sequence:

```bash
packer fmt -check .
packer validate .
packer build .
```

Protect:

- Cloud credentials
- Backend configuration
- Terraform state
- Variable files
- Plan files
- IAM instance profiles
- Temporary build instances

Never print sensitive files or the complete environment in logs.

For private EC2 builds, verify SSM Agent status, IAM permissions, NAT/VPC endpoint access, and instance connectivity.

---

## 20. SSH Agents, Bastions and SSM

SSH diagnostics:

```bash
ssh -vvv user@host
nc -vz host 22
getent hosts host
```

Bastion pattern:

```text
Jenkins Agent
     |
     v
Bastion Host
     |
     v
Private EC2 Instance
```

Example:

```bash
ssh -J bastion-user@bastion target-user@private-host
```

AWS Systems Manager can avoid inbound SSH when:

- SSM Agent is installed and running
- Instance IAM permissions are correct
- Network access to SSM endpoints exists
- The instance is online in Systems Manager

For private instances, NAT or VPC interface endpoints may be required.

---

## 21. CI Secrets and Credentials

Never expose secrets through:

```bash
echo "$PASSWORD"
env
set
ps aux
```

Command-line arguments may be visible through process inspection.

Prefer:

- Jenkins credential bindings
- Secret files with mode `600`
- Secret managers
- Short-lived cloud credentials
- IAM roles/instance profiles
- OIDC federation

Check permissions:

```bash
stat secret-file
chmod 600 secret-file
```

Do not store secrets in Git, workspaces, artifacts, logs, Terraform state without protection, or temporary files.

---

## 22. Common Linux CI/CD Failures

### `command not found`

```bash
command -v terraform
command -v mvn
command -v node
echo "$PATH"
```

The tool may be installed for one user but unavailable to the Jenkins service user.

### Permission denied

```bash
id
ls -ld .
ls -l file
namei -l /path/to/file
```

Inspect every parent directory. Also consider ACLs, SELinux/AppArmor, and read-only filesystems.

### Port already in use

```bash
ss -lntp
sudo lsof -i :8080
fuser -v 8080/tcp
```

### Disk full

```bash
df -h
df -i
du -xhd1 /var | sort -h
```

Check both blocks and inodes.

### Out of memory

```bash
free -h
dmesg -T | grep -i -E 'oom|killed process'
journalctl -k | grep -i oom
```

### Network/DNS problem

```bash
getent hosts repo.example.com
curl -I https://repo.example.com
nc -vz repo.example.com 443
```

### Manual build works, Jenkins fails

Compare:

```bash
whoami
id
pwd
env | sort
echo "$PATH"
java -version
mvn -version
ulimit -a
```

The Jenkins service environment is usually different from an interactive shell.

---

## 23. Production-Grade Pipeline Shell Template

```bash
#!/usr/bin/env bash
set -euo pipefail

log() {
    printf '[%s] %s
' "$(date -Is)" "$*"
}

cleanup() {
    log "Cleaning temporary resources"
}

trap cleanup EXIT INT TERM

: "${WORKSPACE:?WORKSPACE is required}"
: "${BUILD_NUMBER:?BUILD_NUMBER is required}"

log "User: $(id -un)"
log "Host: $(hostname)"
log "Workspace: ${WORKSPACE}"
log "Build: ${BUILD_NUMBER}"

command -v git
command -v java
command -v mvn

git rev-parse --verify HEAD
java -version
mvn -version

mvn -B clean verify

artifact="$(find target -maxdepth 1 -type f -name '*.jar' | head -n 1)"

if [ -z "$artifact" ]; then
    log "ERROR: artifact was not generated"
    exit 1
fi

log "Artifact: $artifact"
sha256sum "$artifact"
log "Build completed successfully"
```

---

## 24. Example Jenkinsfile

```groovy
pipeline {
    agent { label 'linux-java17' }

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        MAVEN_OPTS = '-Xms256m -Xmx1024m'
    }

    stages {
        stage('Diagnostics') {
            steps {
                sh '''
                    set -euo pipefail
                    whoami
                    id
                    pwd
                    df -h
                    free -h
                    java -version
                    mvn -version
                '''
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
                sh 'git rev-parse HEAD'
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    set -euo pipefail
                    mvn -B clean verify
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }
    }

    post {
        always {
            sh '''
                set +e
                echo "Final disk usage:"
                df -h
            '''
            cleanWs()
        }
    }
}
```

---

## 25. Shared Library Considerations

Shared libraries centralize pipeline logic:

```text
vars/
  buildJavaApp.groovy
  deployAnsible.groovy

src/
  org/company/ci/
    Utilities.groovy

resources/
  templates/
```

Linux-related risks:

- Script assumes a specific path
- Tool exists on only one agent
- Shell differs between agents
- Permissions differ
- Environment variables are missing
- Workspace path contains spaces
- Script assumes `/bin/bash`

Define tool requirements and validate them early.

---

## 26. Security Checklist

- [ ] Builds run as non-root users
- [ ] Untrusted code uses isolated agents
- [ ] Sudo is restricted
- [ ] Jenkins directories are protected
- [ ] Private keys are not stored in workspaces
- [ ] Secrets are masked
- [ ] Operating system is patched
- [ ] Inbound network access is restricted
- [ ] Cloud credentials are short-lived
- [ ] Temporary files are cleaned
- [ ] Workspaces do not cross-contaminate
- [ ] Resource usage is monitored
- [ ] Agent images are reproducible
- [ ] Pipeline code changes are reviewed
- [ ] Terraform backend/state is protected
- [ ] Downloaded tools are verified
- [ ] Application services do not run as root

---

## 27. Interview Questions

### Q1. Why should Jenkins builds not run as root?

Build scripts execute arbitrary commands from source code. Root execution increases the blast radius of malicious or faulty commands. A dedicated service account follows least privilege.

### Q2. Why does a command work manually but fail in Jenkins?

The environments differ: user, `PATH`, `HOME`, `JAVA_HOME`, permissions, working directory, shell, `ulimit`, credentials, and network access.

### Q3. How do you troubleshoot `mvn: command not found`?

```bash
whoami
echo "$PATH"
command -v mvn
java -version
mvn -version
```

Then verify the agent configuration and service environment.

### Q4. Why can too many executors reduce performance?

Builds compete for CPU, memory, disk I/O, network bandwidth, ports, and dependency caches.

### Q5. What is the difference between `npm install` and `npm ci`?

`npm ci` is intended for clean, reproducible CI installation using the lock file.

### Q6. How do you investigate port 8080 already being occupied?

```bash
sudo lsof -i :8080
ss -lntp
ps -fp <PID>
```

Identify the owner and clean it up through traps or service management.

### Q7. How do you validate an artifact?

Check existence, size, file type, checksum, archive contents, and optionally startup/health.

### Q8. Why use `set -euo pipefail`?

It exposes command failures, unset variables, and failures hidden inside pipelines.

### Q9. Why is `chmod -R 777` a bad fix?

It grants excessive permissions, hides ownership mistakes, and may expose credentials.

### Q10. How do you deploy a systemd application safely?

Validate the artifact, install it with correct ownership and mode, validate configuration, restart/reload, inspect logs, and run a health check.

### Q11. What causes `Permission denied` in CI?

Wrong owner, missing write permission, parent directory restrictions, ACLs, SELinux/AppArmor, or a read-only filesystem.

### Q12. How do you troubleshoot disk failures?

```bash
df -h
df -i
du -xhd1 /var/lib/jenkins | sort -h
```

### Q13. Why should artifacts be immutable?

They make releases traceable, reproducible, and easier to roll back.

### Q14. What is the controller/agent difference?

The controller coordinates; the agent executes.

### Q15. How can CI deploy to private EC2 without SSH?

Use Systems Manager when the SSM Agent, IAM permissions, and network connectivity are correctly configured.

---

## 28. Troubleshooting Decision Tree

```text
Pipeline failed
     |
     v
Read the first meaningful error
     |
     +--> command not found
     |       |
     |       +--> check PATH, tool installation, agent label
     |
     +--> permission denied
     |       |
     |       +--> check user, owner, mode, parent directories, SELinux
     |
     +--> connection refused/timeout
     |       |
     |       +--> check DNS, routes, security groups, listener
     |
     +--> disk full
     |       |
     |       +--> df -h, df -i, du, logs, caches
     |
     +--> Java/Maven failure
     |       |
     |       +--> java -version, mvn -version, JAVA_HOME, toolchain
     |
     +--> port conflict
     |       |
     |       +--> ss, lsof, stale process cleanup
     |
     +--> artifact missing
             |
             +--> inspect output, permissions, paths, cleanup timing
```

---

## 29. Command Reference

### Identity and environment

```bash
whoami
id
hostname
pwd
env | sort
echo "$PATH"
ulimit -a
```

### Processes

```bash
ps -ef
pgrep -af java
top
htop
kill <PID>
```

### Files and permissions

```bash
ls -lah
stat file
namei -l /path
find . -type f
du -sh .
```

### Disk and memory

```bash
df -h
df -i
free -h
vmstat 1 5
iostat -xz 1 5
```

### Network

```bash
getent hosts example.com
curl -I https://example.com
nc -vz host 443
ss -lntp
```

### Services and logs

```bash
systemctl status service
systemctl restart service
systemctl is-active service
journalctl -u service -n 100 --no-pager
```

### Build tools

```bash
git --version
java -version
mvn -version
node --version
npm --version
python3 --version
go version
terraform version
packer version
ansible --version
```

---

## 30. Final DevOps Checklist

Before blaming application code, verify:

- [ ] Correct agent selected
- [ ] Correct Linux user
- [ ] Correct workspace
- [ ] Correct branch/commit
- [ ] Correct Java/Python/Node/Go version
- [ ] Required tool exists in `PATH`
- [ ] Dependency repository is reachable
- [ ] DNS resolution works
- [ ] Disk space is available
- [ ] Inodes are available
- [ ] Memory is sufficient
- [ ] File ownership is correct
- [ ] Build cache is not corrupted
- [ ] No stale process owns required ports
- [ ] Secrets are available securely
- [ ] Artifact was generated
- [ ] Artifact checksum was recorded
- [ ] Deployment target is reachable
- [ ] systemd/NGINX configuration validates
- [ ] Health check passes
- [ ] Logs are available for rollback analysis

---

## Key Takeaway

A CI/CD engineer must understand Linux as an execution platform, not merely as an operating system.

The most useful question is:

> **What command ran, as which user, on which host, inside which directory, with which environment, using which resources, and what did Linux report?**

That question resolves a large percentage of real-world CI/CD failures.
