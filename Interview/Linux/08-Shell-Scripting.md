# Shell Scripting and Bash Automation for DevOps

## Scope

This guide covers Bash scripting from a **DevOps Engineer's perspective**.

The focus is on:

- Automating deployments
- Writing reliable CI/CD scripts
- Handling errors safely
- Working with files, processes and services
- Using scripts with Jenkins, Ansible, Packer and EC2
- Debugging production automation
- Making scripts idempotent and maintainable

---

# 1. Why is Bash important for DevOps?

Bash is commonly used to automate:

- Package installation
- Server bootstrap
- Application deployment
- Log collection
- Backup tasks
- Health checks
- Build steps
- Service restarts
- Infrastructure validation
- Incident-response commands

Examples:

```bash
#!/usr/bin/env bash

echo "Starting deployment"
sudo systemctl restart myapp
curl --fail http://127.0.0.1:8080/health
```

Bash is useful for short operational workflows. For complex configuration management, prefer tools such as Ansible, Terraform or a dedicated programming language.

---

# 2. What is a shebang?

The shebang specifies which interpreter should execute the script.

Preferred:

```bash
#!/usr/bin/env bash
```

Alternative:

```bash
#!/bin/bash
```

Make the script executable:

```bash
chmod +x deploy.sh
```

Run it:

```bash
./deploy.sh
```

Or explicitly:

```bash
bash deploy.sh
```

## Difference

```bash
./deploy.sh
```

uses the interpreter specified by the shebang.

```bash
bash deploy.sh
```

explicitly runs Bash regardless of the shebang.

---

# 3. Why use `set -euo pipefail`?

A common safety baseline is:

```bash
set -euo pipefail
```

It enables:

- `-e`: Exit when a command fails in many contexts
- `-u`: Treat unset variables as errors
- `pipefail`: Make a pipeline fail if an earlier command fails

Example:

```bash
#!/usr/bin/env bash
set -euo pipefail

mkdir -p /opt/myapp
cp app.jar /opt/myapp/
sudo systemctl restart myapp
```

## Important limitation

`set -e` is not a complete error-handling system. Its behavior has exceptions around:

- `if` conditions
- `while` and `until`
- `&&` and `||`
- Pipelines
- Command substitutions

Use explicit checks for critical operations.

---

# 4. Use explicit error handling

Example:

```bash
if ! sudo systemctl restart myapp; then
  echo "ERROR: Failed to restart myapp" >&2
  exit 1
fi
```

Health check:

```bash
if ! curl --fail --silent --show-error http://127.0.0.1:8080/health; then
  echo "ERROR: Application health check failed" >&2
  exit 1
fi
```

A useful error function:

```bash
die() {
  echo "ERROR: $*" >&2
  exit 1
}
```

Use it:

```bash
[[ -f app.jar ]] || die "Application artifact not found"
```

---

# 5. What is an exit status?

Every command returns an exit status.

- `0`: Success
- Non-zero: Failure or special condition

Example:

```bash
true
echo $?

false
echo $?
```

Capture a status:

```bash
if command; then
  echo "Success"
else
  echo "Failure"
fi
```

Do not do this when you need the status of a specific command:

```bash
command
echo "Finished"
```

The script may continue even if `command` failed.

In CI/CD, the final exit code often determines whether a stage succeeds.

---

# 6. Variables

Define variables:

```bash
APP_NAME="salary-api"
APP_PORT=8080
ENVIRONMENT="dev"
```

Use variables:

```bash
echo "$APP_NAME"
echo "$APP_PORT"
```

Always quote variables unless word splitting or glob expansion is intentionally required:

```bash
rm -- "$file"
cp -- "$source" "$destination"
```

Avoid:

```bash
rm $file
```

because spaces, wildcard characters or empty values can cause unexpected behavior.

---

# 7. Quoting rules

## Double quotes

Variables expand:

```bash
echo "$HOME"
```

## Single quotes

Variables do not expand:

```bash
echo '$HOME'
```

## No quotes

Word splitting and glob expansion may occur:

```bash
echo $HOME
```

## DevOps rule

Use double quotes around paths and variables:

```bash
tar -czf "$backup_file" "$source_dir"
```

---

# 8. Command substitution

Use `$()` to capture command output:

```bash
CURRENT_DATE="$(date +%F)"
HOSTNAME_VALUE="$(hostname)"
PID="$(pgrep -f 'java -jar app.jar' | head -n 1)"
```

Avoid old-style backticks:

```bash
`date`
```

Prefer:

```bash
$(date)
```

because it is easier to nest and read.

---

# 9. Conditions

File checks:

```bash
if [[ -f "$file" ]]; then
  echo "Regular file exists"
fi
```

Common tests:

| Test | Meaning |
|---|---|
| `-f` | Regular file |
| `-d` | Directory |
| `-e` | Path exists |
| `-r` | Readable |
| `-w` | Writable |
| `-x` | Executable |
| `-s` | Non-empty file |
| `-z` | Empty string |
| `-n` | Non-empty string |

String comparison:

```bash
if [[ "$ENVIRONMENT" == "production" ]]; then
  echo "Production deployment"
fi
```

Numeric comparison:

```bash
if (( APP_PORT == 8080 )); then
  echo "Expected port"
fi
```

---

# 10. Loops

## For loop

```bash
for server in app01 app02 app03; do
  echo "Checking $server"
done
```

## While loop

```bash
attempt=1

while (( attempt <= 5 )); do
  echo "Attempt $attempt"
  ((attempt++))
done
```

## Read a file safely

```bash
while IFS= read -r line; do
  echo "$line"
done < servers.txt
```

Avoid:

```bash
for line in $(cat servers.txt); do
  echo "$line"
done
```

The latter breaks on whitespace and special characters.

---

# 11. Functions

Functions make scripts reusable.

```bash
log() {
  printf '[%s] %s\n' "$(date '+%F %T')" "$*"
}

check_service() {
  local service_name="$1"

  systemctl is-active --quiet "$service_name"
}
```

Use:

```bash
log "Checking application"
if check_service myapp; then
  log "Application is active"
fi
```

Use `local` for function variables to avoid accidental global changes.

---

# 12. Positional parameters

Example:

```bash
#!/usr/bin/env bash
set -euo pipefail

environment="${1:-dev}"
version="${2:-latest}"

echo "Environment: $environment"
echo "Version: $version"
```

Run:

```bash
./deploy.sh production 1.4.2
```

Useful parameters:

| Parameter | Meaning |
|---|---|
| `$0` | Script name |
| `$1`, `$2` | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments, preserving boundaries when quoted |
| `$?` | Previous command status |
| `$$` | Current shell PID |
| `$!` | PID of last background process |

Prefer:

```bash
for arg in "$@"; do
  echo "$arg"
done
```

---

# 13. Validate script arguments

Example:

```bash
usage() {
  echo "Usage: $0 <environment> <version>"
}

if [[ $# -ne 2 ]]; then
  usage
  exit 2
fi

environment="$1"
version="$2"
```

Use meaningful exit codes:

- `0`: Success
- `1`: General failure
- `2`: Invalid command usage

Never allow an empty environment or version to silently trigger a production deployment.

---

# 14. `getopts` for command-line options

Example:

```bash
environment="dev"
dry_run=false

while getopts ":e:n" opt; do
  case "$opt" in
    e) environment="$OPTARG" ;;
    n) dry_run=true ;;
    *)
      echo "Usage: $0 [-e environment] [-n]"
      exit 2
      ;;
  esac
done

echo "Environment: $environment"
echo "Dry run: $dry_run"
```

Run:

```bash
./deploy.sh -e production -n
```

This is useful for operational scripts that need flags.

---

# 15. Temporary files and cleanup

Use `mktemp`:

```bash
tmp_file="$(mktemp)"
```

Clean it up:

```bash
rm -f -- "$tmp_file"
```

A cleanup trap:

```bash
tmp_dir="$(mktemp -d)"

cleanup() {
  rm -rf -- "$tmp_dir"
}

trap cleanup EXIT
```

Other useful traps:

```bash
trap 'echo "Interrupted" >&2; exit 130' INT TERM
```

Be careful with `rm -rf`. Validate the variable before deleting:

```bash
[[ -n "$tmp_dir" ]] || exit 1
[[ "$tmp_dir" == /tmp/* ]] || exit 1
rm -rf -- "$tmp_dir"
```

---

# 16. Logging from scripts

Use timestamps and stderr for errors:

```bash
log() {
  printf '[%s] INFO: %s\n' "$(date '+%F %T')" "$*"
}

error() {
  printf '[%s] ERROR: %s\n' "$(date '+%F %T')" "$*" >&2
}
```

Example:

```bash
log "Starting deployment"
error "Health check failed"
```

In Jenkins, logs should clearly identify:

- Environment
- Version
- Host
- Stage
- Command being executed
- Failure reason
- Rollback action

Never print secrets.

---

# 17. Redirecting output

Overwrite a file:

```bash
command > output.log
```

Append:

```bash
command >> output.log
```

Redirect stderr:

```bash
command 2> error.log
```

Redirect both:

```bash
command > output.log 2>&1
```

Modern Bash shorthand:

```bash
command &> output.log
```

Pipe output:

```bash
command | tee output.log
```

Append with `tee`:

```bash
command | tee -a output.log
```

Be careful: a pipeline's exit status may hide failures unless `pipefail` is enabled.

---

# 18. Safe use of `grep`, `awk` and `sed`

## grep

```bash
grep -F "ERROR" app.log
grep -E 'timeout|connection refused' app.log
```

## awk

```bash
awk '{print $1, $5}' access.log
```

## sed

```bash
sed -n '1,20p' app.log
sed 's/old-value/new-value/g' config.txt
```

Avoid modifying production configuration blindly with broad substitutions.

Before changing a file:

```bash
cp config.yaml "config.yaml.$(date +%s).bak"
```

Prefer templating tools such as Ansible for managed configuration.

---

# 19. Check command availability

```bash
require_command() {
  command -v "$1" >/dev/null 2>&1 || {
    echo "Missing required command: $1" >&2
    exit 1
  }
}

require_command curl
require_command systemctl
require_command jq
```

This makes failures clear instead of allowing the script to fail later with a confusing message.

---

# 20. Wait for a service or endpoint

A retry loop:

```bash
url="http://127.0.0.1:8080/health"

for attempt in {1..30}; do
  if curl --fail --silent "$url" >/dev/null; then
    echo "Application is ready"
    exit 0
  fi

  echo "Waiting for application: attempt $attempt"
  sleep 2
done

echo "Application did not become ready" >&2
exit 1
```

Use timeouts:

```bash
curl --connect-timeout 3 --max-time 5 --fail "$url"
```

Avoid infinite retry loops in CI/CD unless they have an explicit timeout.

---

# 21. Idempotency in Bash

An idempotent script can be run repeatedly without causing unwanted changes.

Bad:

```bash
echo "PORT=8080" >> /etc/myapp.env
```

Every run adds another line.

Better:

```bash
grep -q '^PORT=' /etc/myapp.env || echo 'PORT=8080' >> /etc/myapp.env
```

For complex configuration, use Ansible templates instead of repeatedly editing files with shell commands.

Other examples:

```bash
mkdir -p /opt/myapp
systemctl enable myapp
```

These are generally safe to repeat.

---

# 22. Avoid parsing `ps` output when possible

Fragile:

```bash
ps aux | grep java | awk '{print $2}'
```

Better:

```bash
pgrep -f 'java -jar app.jar'
```

Or use systemd:

```bash
systemctl show myapp -p MainPID --value
```

If you must parse command output, make the pattern precise and handle no-match cases.

---

# 23. Running commands remotely

Example with SSH:

```bash
ssh -o BatchMode=yes ubuntu@server \
  'sudo systemctl restart myapp'
```

Use strict failure handling:

```bash
if ! ssh -o BatchMode=yes ubuntu@server \
    'sudo systemctl is-active --quiet myapp'; then
  echo "Remote service validation failed" >&2
  exit 1
fi
```

For cloud environments, consider:

- AWS Systems Manager
- Ansible
- SSM Run Command
- Configuration-management pipelines

Avoid embedding passwords in scripts.

---

# 24. Bash in Jenkins

Example Jenkins shell step:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Building application"
./mvnw clean package

echo "Validating artifact"
test -f target/app.jar

echo "Deployment completed"
```

Good practices:

- Fail fast on errors
- Use explicit paths
- Validate artifacts
- Do not print secrets
- Use credentials bindings
- Keep deployment scripts version-controlled
- Return non-zero on failure
- Publish useful logs

---

# 25. Bash in Packer

A Packer shell provisioner may use:

```bash
#!/usr/bin/env bash
set -euo pipefail

sudo apt-get update
sudo apt-get install -y curl unzip nginx

sudo systemctl enable nginx

nginx -v
curl --version
```

Image provisioning should be:

- Repeatable
- Non-interactive
- Validated
- Free of temporary secrets
- Cleaned up before image creation

Do not leave cloud credentials, private keys or sensitive build files inside the image.

---

# 26. Bash in Ansible

Ansible modules are preferred when they exist.

Less desirable:

```yaml
- name: Install nginx
  ansible.builtin.shell: apt-get install -y nginx
```

Better:

```yaml
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true
  become: true
```

Use shell commands when:

- A module cannot express the operation
- A vendor installer is required
- A diagnostic command is needed
- A carefully controlled migration is being performed

When using `shell`, define:

- `creates`
- `removes`
- `changed_when`
- `failed_when`

Example:

```yaml
- name: Run migration once
  ansible.builtin.shell: /opt/myapp/migrate.sh
  args:
    creates: /opt/myapp/.migration-complete
```

---

# 27. ShellCheck and formatting

Install ShellCheck:

```bash
sudo apt-get install shellcheck
```

Run:

```bash
shellcheck deploy.sh
```

ShellCheck catches many problems involving:

- Quoting
- Variables
- Word splitting
- Incorrect tests
- Unsafe command substitutions
- Common Bash mistakes

Format scripts consistently. Use meaningful names and avoid unnecessary cleverness.

---

# 28. Debugging Bash scripts

Run with tracing:

```bash
bash -x deploy.sh
```

Enable tracing inside a script:

```bash
set -x
```

Disable:

```bash
set +x
```

Show variables safely:

```bash
printf 'environment=%q\n' "$environment"
```

Avoid tracing commands that contain secrets. In CI/CD, disable tracing around sensitive commands.

Use:

```bash
set +x
# secret operation
set -x
```

---

# 29. Scenario: Script succeeds manually but fails in Jenkins

Compare:

- Current directory
- User identity
- PATH
- Environment variables
- Available credentials
- Shell interpreter
- File permissions
- Non-interactive behavior

Diagnostics:

```bash
whoami
pwd
id
env | sort
command -v java
java -version
```

Do not print secret environment variables.

Use absolute paths and explicitly configure required environment variables.

---

# 30. Scenario: Deployment script reports success although deployment failed

Likely causes:

- Failed command was not checked
- Script did not use `set -e`
- Pipeline hid an earlier failure
- Final command returned `0`
- Health check was missing
- Error was only printed, not returned

Improve:

```bash
set -euo pipefail

sudo systemctl restart myapp
curl --fail http://127.0.0.1:8080/health
```

For critical commands:

```bash
if ! deploy_artifact; then
  echo "Deployment failed" >&2
  exit 1
fi
```

---

# 31. Scenario: Script deletes the wrong directory

Risks often come from:

- Unset variables
- Incorrect path construction
- Missing quotes
- Unexpected wildcard expansion
- Running as root

Safer:

```bash
set -u

release_dir="${1:-}"

[[ -n "$release_dir" ]] || {
  echo "Release directory is required" >&2
  exit 2
}

[[ "$release_dir" == /opt/myapp/releases/* ]] || {
  echo "Refusing unsafe path: $release_dir" >&2
  exit 1
}

rm -rf -- "$release_dir"
```

Test destructive scripts in a non-production directory first.

---

# 32. Scenario: Script hangs forever

Possible causes:

- Waiting for a service that never becomes ready
- SSH command waiting for input
- Package manager prompt
- Network request without timeout
- Deadlock in a loop
- Missing command output handling

Use:

```bash
timeout 60s ./deploy.sh
```

For network calls:

```bash
curl --connect-timeout 5 --max-time 15 URL
```

For SSH:

```bash
ssh -o BatchMode=yes -o ConnectTimeout=10 user@host command
```

Every external dependency should have a reasonable timeout.

---

# 33. Scenario: Parallel execution

Run a background task:

```bash
long_task &
pid=$!

wait "$pid"
```

Run several tasks:

```bash
pids=()

for host in app01 app02 app03; do
  check_host "$host" &
  pids+=("$!")
done

status=0

for pid in "${pids[@]}"; do
  wait "$pid" || status=1
done

exit "$status"
```

Parallelism can reduce execution time, but consider:

- API rate limits
- Host capacity
- Ordering requirements
- Log interleaving
- Failure handling
- Deployment blast radius

---

# 34. Scenario: Bash script receives a secret

Avoid:

```bash
echo "Password=$PASSWORD"
```

Avoid command-line secrets when possible because arguments may be visible through process inspection.

Prefer:

- Jenkins credentials binding
- AWS Secrets Manager
- SSM Parameter Store
- Restricted files
- Short-lived credentials
- IAM roles

If a secret must be read interactively:

```bash
read -r -s password
printf '\n'
```

Never commit secrets into Git.

---

# 35. Useful Bash command reference

| Requirement | Command |
|---|---|
| Run script | `bash script.sh` |
| Make executable | `chmod +x script.sh` |
| Trace script | `bash -x script.sh` |
| Check syntax | `bash -n script.sh` |
| Find command | `command -v command` |
| Exit status | `echo $?` |
| Create temp file | `mktemp` |
| Cleanup on exit | `trap cleanup EXIT` |
| Search text | `grep -F pattern file` |
| Transform text | `sed`, `awk` |
| Check file | `[[ -f file ]]` |
| Check service | `systemctl is-active service` |
| HTTP health check | `curl --fail URL` |
| Timeout command | `timeout 60s command` |
| Shell linting | `shellcheck script.sh` |
| Read arguments | `"$@"` |
| Current script path | `${BASH_SOURCE[0]}` |

---

# 36. Interview questions

1. What is a shebang?
2. Why use `set -euo pipefail`?
3. What are the limitations of `set -e`?
4. Why should variables usually be quoted?
5. What is the difference between `$@` and `$*`?
6. How do you validate script arguments?
7. What is an exit status?
8. How do you handle errors explicitly?
9. What is command substitution?
10. How do you create and clean up temporary files?
11. What is a trap?
12. How do you write an idempotent Bash script?
13. Why is parsing `ps | grep` fragile?
14. How do you add timeouts to scripts?
15. How do you troubleshoot a script that works manually but fails in Jenkins?
16. How do you prevent secrets from appearing in logs?
17. When should Bash be replaced with Ansible or Python?
18. How do you debug a Bash script?
19. How do you run commands in parallel safely?
20. How do you use Bash in Packer?
21. How do you use Bash in Ansible?
22. Why is `shell` less desirable than an Ansible module?
23. How do you verify a deployment from a Bash script?
24. How do you prevent unsafe `rm -rf` operations?
25. How do you troubleshoot a script that hangs?

---

# 37. Interview checklist

- [ ] Write a safe Bash script
- [ ] Use a correct shebang
- [ ] Use `set -euo pipefail`
- [ ] Quote variables
- [ ] Validate arguments
- [ ] Use functions
- [ ] Handle exit codes
- [ ] Use traps and temporary files
- [ ] Add timeouts
- [ ] Implement retries
- [ ] Write idempotent operations
- [ ] Use `grep`, `awk` and `sed` safely
- [ ] Debug with `bash -x`
- [ ] Validate with ShellCheck
- [ ] Use Bash in Jenkins
- [ ] Use Bash in Packer
- [ ] Know when to use Ansible instead
- [ ] Avoid exposing secrets
- [ ] Validate application health after deployment

---

# Key DevOps principle

**A deployment script is production code.**

It must be:

- Predictable
- Idempotent where possible
- Safe with paths and variables
- Explicit about failures
- Protected against hanging
- Free of secrets in logs
- Validated before declaring success

A script that “usually works” is not reliable automation.
