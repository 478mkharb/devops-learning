# Linux for DevOps — Topic 9: Environment Variables and Configuration Management

## Scope

This topic explains how Linux applications receive configuration and how DevOps tools manage it across laptops, Jenkins, Ansible, Packer, Terraform, and EC2.

The key distinction is:

> **Code defines behavior. Configuration selects behavior for an environment. Secrets must be protected separately.**

---

# 1. What is an environment variable?

An environment variable is a key-value pair made available to a process.

```bash
APP_ENV=dev
PORT=8080
LOG_LEVEL=info
```

Applications commonly use environment variables for:

- Environment selection: `dev`, `test`, `prod`
- Port numbers
- Database hostnames and ports
- Feature flags
- Log levels
- Cloud regions
- Credentials and tokens
- Paths to configuration files

View variables:

```bash
env
printenv
printenv HOME
echo "$HOME"
```

Do not expose secrets casually:

```bash
env | grep -i password
history
ps auxww
```

Some secrets can appear in process arguments, shell history, CI logs, or crash reports.

---

# 2. Shell variables vs environment variables

## Local shell variable

```bash
APP_ENV=dev
echo "$APP_ENV"
```

This variable exists in the current shell but is not automatically inherited by child processes.

```bash
APP_ENV=dev
bash -c 'echo "$APP_ENV"'
```

Output is usually empty.

## Exported environment variable

```bash
export APP_ENV=dev
bash -c 'echo "$APP_ENV"'
```

Output:

```text
dev
```

An exported variable is inherited by child processes.

## One-command environment variable

```bash
APP_ENV=prod ./start-app.sh
```

The variable is available to that command only.

This is useful in deployments:

```bash
APP_ENV=staging ./deploy.sh
```

## Remove a variable

```bash
unset APP_ENV
```

## List shell variables and functions

```bash
set
```

`set` may show much more than environment variables. Use `env` or `printenv` when you specifically need exported variables.

---

# 3. Important Linux variables

| Variable | Meaning |
|---|---|
| `PATH` | Directories searched for executable commands |
| `HOME` | User's home directory |
| `PWD` | Current working directory |
| `OLDPWD` | Previous working directory |
| `SHELL` | User's configured shell |
| `USER` | Current username in many environments |
| `LOGNAME` | Login name |
| `LANG` | Locale |
| `HOSTNAME` | Hostname, if provided |
| `EDITOR` | Preferred editor |
| ` umask` | Default permission mask; commonly viewed with `umask` |

Inspect:

```bash
printf 'USER=%s\nHOME=%s\nPATH=%s\n' "$USER" "$HOME" "$PATH"
```

Never assume every variable exists in every execution context.

---

# 4. PATH and command resolution

Linux searches the directories in `PATH` from left to right.

```bash
echo "$PATH"
command -v java
command -v python3
type -a python3
```

Temporarily add a directory:

```bash
export PATH="$HOME/bin:$PATH"
```

Prefer this form:

```bash
export PATH="/opt/myapp/bin:$PATH"
```

Avoid:

```bash
export PATH="/opt/myapp/bin"
```

The second command can remove standard directories and break commands such as `sudo`, `systemctl`, or `ls`.

Check the actual executable:

```bash
readlink -f "$(command -v java)"
```

A frequent DevOps problem is:

> It works in my SSH session but fails in Jenkins or systemd.

Usually, those processes receive a different `PATH`.

Use absolute paths in production service definitions where practical:

```ini
ExecStart=/usr/bin/python3 /opt/myapp/app.py
```

---

# 5. Where variables are configured

## `/etc/environment`

System-wide environment configuration for login-related tooling.

Example:

```text
APP_ENV=prod
APP_HOME=/opt/myapp
```

Do not treat this file as a shell script. Shell syntax such as command substitution is not generally appropriate here.

## `/etc/profile`

System-wide login-shell configuration.

## `/etc/profile.d/*.sh`

Preferred location for modular system-wide shell configuration.

Example:

```bash
sudo tee /etc/profile.d/myapp.sh >/dev/null <<'EOF'
export APP_HOME=/opt/myapp
export PATH="$APP_HOME/bin:$PATH"
EOF
```

## `~/.profile`

Per-user login-shell configuration. Often used to configure user environment variables.

## `~/.bashrc`

Per-user interactive Bash configuration. Commonly used for aliases, prompt configuration, and interactive settings.

Do not put critical production application configuration only in `~/.bashrc`.

---

# 6. Login vs non-login and interactive vs non-interactive shells

These are different concepts.

| Shell type | Typical example |
|---|---|
| Login shell | User logs in through SSH |
| Non-login shell | `bash` started from another shell |
| Interactive shell | User can type commands |
| Non-interactive shell | Script executed by a scheduler or service |

Examples:

```bash
ssh ubuntu@server
bash
bash -c 'env'
bash script.sh
```

A Jenkins shell, cron job, systemd service, and SSH session may load different configuration files.

Debug the shell:

```bash
ps -p $$ -o pid,ppid,args=
echo "$0"
shopt -q login_shell && echo login || echo non-login
case $- in *i*) echo interactive ;; *) echo non-interactive ;; esac
```

Best practice:

- Do not rely on interactive shell startup files for services.
- Make service configuration explicit.
- Use `/etc/profile.d` only for human login environments.
- Use systemd `EnvironmentFile`, Ansible templates, or a dedicated config file for applications.

---

# 7. Why systemd does not see your SSH environment

Suppose this works:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
./run-app.sh
```

But systemd fails:

```text
java: command not found
```

The systemd service is not running inside your SSH shell. It normally does not load your user's `.bashrc` or `.profile`.

Use a service definition:

```ini
[Service]
Environment="APP_ENV=prod"
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"
ExecStart=/opt/myapp/bin/start.sh
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
sudo systemctl show myapp --property=Environment
```

---

# 8. systemd environment configuration

## Inline `Environment`

```ini
[Service]
Environment="APP_ENV=prod"
Environment="PORT=8080"
```

## `EnvironmentFile`

```ini
[Service]
EnvironmentFile=/etc/myapp/myapp.env
```

Example file:

```text
APP_ENV=prod
PORT=8080
LOG_LEVEL=info
```

Protect it if it contains secrets:

```bash
sudo chown root:root /etc/myapp/myapp.env
sudo chmod 600 /etc/myapp/myapp.env
```

Optional file:

```ini
EnvironmentFile=-/etc/myapp/optional.env
```

The leading `-` means the service does not fail merely because the file is missing.

## `PassEnvironment`

This is relevant when variables exist in the system manager environment and need to be passed into a service. Do not assume it forwards your SSH shell variables.

## `UnsetEnvironment`

Use it to remove variables that should not reach the process.

Inspect the final service configuration:

```bash
systemctl cat myapp
systemctl show myapp
systemctl show myapp --property=Environment
```

---

# 9. Configuration, secrets, and code

Separate these concerns:

| Category | Examples | Recommended handling |
|---|---|---|
| Code | Application source, scripts | Git |
| Non-secret config | Port, log level, feature flags | Versioned config/templates |
| Environment selection | `dev`, `staging`, `prod` | Deployment variables |
| Secrets | Passwords, API keys, tokens | Secret manager, CI credentials, Ansible Vault |
| Runtime state | PID files, caches, generated data | Dedicated runtime directories |

Never commit:

```text
.env
passwords.yml
private.key
cloud-credentials
```

Use a safe template:

```text
# .env.example
APP_ENV=dev
DB_HOST=localhost
DB_PORT=5432
DB_USER=appuser
DB_PASSWORD=replace-me
```

The real file should be supplied securely during deployment.

---

# 10. `.env` files: useful but limited

A `.env` file is only a convention. Linux does not automatically load it.

Example:

```text
APP_ENV=dev
PORT=8080
```

This does not automatically affect every process.

Possible loading approaches:

```bash
set -a
. ./.env
set +a
```

Be careful: sourcing a file executes shell syntax. Do not source untrusted content.

For systemd, prefer:

```ini
EnvironmentFile=/etc/myapp/myapp.env
```

For CI/CD, inject variables through the CI system's credential and environment mechanisms.

---

# 11. `envsubst` and configuration templates

`envsubst` replaces shell-style variables in text templates.

Template:

```nginx
server {
    listen ${APP_PORT};
    server_name ${APP_HOST};
}
```

Render:

```bash
export APP_PORT=8080
export APP_HOST=example.internal

envsubst < nginx.conf.template > nginx.conf
```

Validate generated configuration before reload:

```bash
nginx -t -c /etc/nginx/nginx.conf
```

Important:

- Validate after substitution.
- Quote values correctly.
- Do not use blind substitution for secrets unless the output file is protected.
- Avoid accidental replacement of variables intended for the application.

---

# 12. Jenkins environment handling

Jenkins has several sources of variables:

- Global node/controller environment
- Pipeline `environment`
- Parameters
- Credentials bindings
- Tool installations
- Shell steps
- Agent-specific configuration

Example:

```groovy
pipeline {
    agent any

    environment {
        APP_ENV = 'staging'
        APP_PORT = '8080'
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''
                    set -eu
                    printf 'Deploying environment: %s\\n' "$APP_ENV"
                    ./deploy.sh
                '''
            }
        }
    }
}
```

Use credentials bindings for secrets rather than hardcoding them:

```groovy
withCredentials([
    string(credentialsId: 'api-token', variable: 'API_TOKEN')
]) {
    sh '''
        set +x
        ./deploy.sh
    '''
}
```

Rules:

- Do not print secrets.
- Avoid `echo "$API_TOKEN"`.
- Be cautious with `set -x`.
- Do not pass secrets in command-line arguments when avoidable.
- Remember that environment variables can be visible to processes running under the same account in some situations.
- Use least-privilege Jenkins agents and credentials.

---

# 13. Ansible variables and configuration

Common variable locations:

```text
group_vars/
host_vars/
inventory
playbook vars
role defaults
role vars
extra vars
```

Example:

```yaml
# group_vars/app_servers.yml
app_env: staging
app_port: 8080
app_home: /opt/myapp
```

Template:

```jinja2
APP_ENV={{ app_env }}
PORT={{ app_port }}
```

Deploy:

```yaml
- name: Install application environment file
  ansible.builtin.template:
    src: myapp.env.j2
    dest: /etc/myapp/myapp.env
    owner: root
    group: root
    mode: '0600'
  notify: Restart myapp
```

Use Ansible Vault for secrets:

```bash
ansible-vault create group_vars/all/vault.yml
ansible-vault edit group_vars/all/vault.yml
ansible-vault view group_vars/all/vault.yml
```

Do not confuse:

- `group_vars`: configuration data
- `host_vars`: host-specific overrides
- Vault-encrypted variables: protected secrets
- Templates: files generated from variables

---

# 14. Packer variables and provisioning

Packer variables describe image-build inputs:

```hcl
variable "aws_region" {
  type    = string
  default = "us-east-1"
}

variable "app_env" {
  type    = string
  default = "prod"
}
```

Use:

```bash
packer build -var="app_env=staging" notification.pkr.hcl
```

Packer image design:

- Install packages during image build.
- Place static application files in predictable paths.
- Avoid baking short-lived secrets into AMIs.
- Use instance boot-time configuration or SSM/secret manager for environment-specific secrets.
- Validate the image after provisioning.

An AMI should be reusable. Avoid making it permanently dependent on one environment unless that is intentional.

---

# 15. Terraform variables vs environment variables

Terraform input variables are not the same as arbitrary Linux environment variables.

Terraform can receive values through:

```text
-var
-var-file
*.tfvars
*.auto.tfvars
TF_VAR_name
```

Example:

```bash
export TF_VAR_aws_region=us-east-1
terraform plan
```

Use environment variables for sensitive or pipeline-supplied values, but remember:

- Terraform state may still contain sensitive values.
- Mark variables as `sensitive = true` to reduce display, not to encrypt state.
- Protect remote state and its access.
- Do not commit secret `.tfvars` files.

Example:

```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```

---

# 16. Configuration precedence and overrides

A common production failure is an unexpected override.

Typical sources include:

1. Built-in application defaults
2. Configuration file
3. Environment variables
4. Command-line arguments
5. CI/CD injected values
6. Runtime orchestration settings

The exact order depends on the application.

Always document:

- Which source wins?
- Which variables are mandatory?
- Which values are safe defaults?
- Which values are environment-specific?
- How are secrets injected?
- Does a change require restart or reload?

Debug effective configuration, not only the source files.

---

# 17. Scenario: works manually but fails in systemd

Symptoms:

```text
systemctl status myapp
# java: command not found
```

Manual session:

```bash
echo "$PATH"
command -v java
./start.sh
```

Diagnosis:

```bash
systemctl show myapp --property=Environment
systemctl cat myapp
journalctl -u myapp -b --no-pager
```

Fix:

```ini
[Service]
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

---

# 18. Scenario: works over SSH but fails in Jenkins

Check:

```bash
whoami
pwd
echo "$PATH"
env | sort
command -v terraform
command -v ansible-playbook
```

Common causes:

- Different user
- Different home directory
- Different PATH
- Different working directory
- Missing SSH agent or credentials
- Missing cloud environment variables
- Different Python virtual environment
- Workspace cleanup
- Non-interactive shell behavior

Use absolute paths or configure Jenkins tools explicitly.

---

# 19. Scenario: wrong environment selected

Symptoms:

- Staging deployment connects to production database.
- Wrong AWS region is used.
- Wrong S3 bucket is selected.
- Application reports `prod` while pipeline says `staging`.

Debug:

```bash
printf 'APP_ENV=%s\n' "${APP_ENV-UNSET}"
printf 'AWS_REGION=%s\n' "${AWS_REGION-UNSET}"
printf 'CONFIG_FILE=%s\n' "${CONFIG_FILE-UNSET}"
```

Add deployment guardrails:

```bash
test "${APP_ENV:-}" = "staging" || {
    echo "Refusing deployment: unexpected APP_ENV"
    exit 1
}
```

Use separate names, accounts, roles, and state locations for environments.

---

# 20. Scenario: secret leaked in logs

Risky:

```bash
set -x
curl -H "Authorization: Bearer $TOKEN" https://api.example.com
```

Safer:

```bash
set +x
curl -fsS \
  -H "Authorization: Bearer ${TOKEN}" \
  https://api.example.com
```

Also:

- Mask credentials in Jenkins.
- Avoid command-line secrets.
- Restrict log access.
- Rotate leaked secrets immediately.
- Review shell history and build artifacts.
- Never place secrets in AMI bake logs or Git.

Masking is not a substitute for proper secret handling.

---

# 21. Scenario: config changed but application did not change

Changing a file does not guarantee that the application reloads it.

Check:

```bash
systemctl status myapp
journalctl -u myapp -n 100 --no-pager
```

Possible actions:

```bash
sudo systemctl reload myapp
sudo systemctl restart myapp
```

Only use `reload` if the application supports it.

For NGINX:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

For applications without reload support, use restart or a controlled rolling deployment.

---

# 22. Scenario: variable contains spaces or special characters

Bad:

```bash
APP_NAME=My App
```

Correct:

```bash
APP_NAME="My App"
export APP_NAME
```

Always quote expansions:

```bash
printf '%s\n' "$APP_NAME"
mkdir -p "$APP_HOME/logs"
```

Avoid unsafe evaluation:

```bash
eval "$USER_INPUT"
```

Use arrays for complex command arguments in Bash:

```bash
args=(--name "$APP_NAME" --port "$PORT")
some-command "${args[@]}"
```

---

# 23. Scenario: configuration differs between EC2 instances

Possible causes:

- Manual changes
- Different AMI versions
- Different Ansible runs
- Different user data
- Different environment files
- Different package versions
- Different IAM roles or instance metadata
- Configuration drift

Compare:

```bash
sha256sum /etc/myapp/myapp.env
rpm -qa | sort
dpkg-query -W | sort
systemctl cat myapp
```

Use configuration management:

- Ansible for desired state
- Packer for repeatable base images
- Terraform for infrastructure
- Git for versioned templates
- CI/CD for controlled promotion

---

# 24. Scenario: rollback and configuration compatibility

A code rollback can fail if configuration has already changed.

Example:

- Version 2 expects `DB_URL`.
- Version 1 expects `DATABASE_HOST`.
- Deployment rolls code back but keeps version-2 configuration.

Use compatibility strategies:

- Support old and new variables during migration.
- Version configuration schemas.
- Validate before deployment.
- Roll back code and compatible configuration together.
- Keep database migrations backward-compatible where possible.

---

# 25. Configuration validation

Validate before restarting production services.

Examples:

```bash
bash -n deploy.sh
python3 -m py_compile app.py
nginx -t
systemd-analyze verify /etc/systemd/system/myapp.service
terraform validate
ansible-playbook --syntax-check site.yml
packer validate notification.pkr.hcl
```

Validate required variables:

```bash
: "${APP_ENV:?APP_ENV is required}"
: "${APP_HOME:?APP_HOME is required}"
```

Validate allowed values:

```bash
case "$APP_ENV" in
  dev|staging|prod) ;;
  *) echo "Invalid APP_ENV"; exit 1 ;;
esac
```

---

# 26. Practical command reference

```bash
env
printenv
printenv PATH
set
export APP_ENV=dev
unset APP_ENV
echo "$PATH"
command -v java
type -a python3
source file.env
envsubst
systemctl show myapp --property=Environment
systemctl cat myapp
systemctl daemon-reload
journalctl -u myapp -b
systemd-analyze verify myapp.service
ansible-vault create secrets.yml
terraform validate
packer validate template.pkr.hcl
```

---

# 27. Interview questions

1. What is the difference between a shell variable and an environment variable?
2. Why must `export` be used?
3. What is the purpose of `PATH`?
4. Why can changing `.bashrc` fail to fix a systemd service?
5. Explain login and non-login shells.
6. Where should system-wide shell variables be configured?
7. How does systemd load an environment file?
8. What is the difference between `Environment` and `EnvironmentFile`?
9. How would you debug a variable missing in Jenkins?
10. How should secrets be passed to Jenkins?
11. Why should secrets not be baked into an AMI?
12. How does Ansible Vault protect sensitive variables?
13. What is the difference between Terraform variables and `TF_VAR_*`?
14. What is configuration drift?
15. How would you prevent staging from deploying to production?
16. Why does changing a config file not always affect a running application?
17. How do you validate a generated NGINX configuration?
18. What happens if you overwrite `PATH` incorrectly?
19. Why are absolute executable paths useful in systemd?
20. How can a code rollback become incompatible with configuration?

---

# 28. Interview checklist

You should be able to:

- [ ] Explain shell vs exported variables.
- [ ] Debug `PATH` problems.
- [ ] Explain `.profile` vs `.bashrc`.
- [ ] Explain why systemd and Jenkins have different environments.
- [ ] Configure `EnvironmentFile` in systemd.
- [ ] Inject non-secret config through Ansible templates.
- [ ] Protect secrets with Jenkins credentials and Ansible Vault.
- [ ] Explain Packer and Terraform variable handling.
- [ ] Validate configuration before deployment.
- [ ] Detect environment mix-ups.
- [ ] Explain configuration drift and rollback compatibility.
- [ ] Avoid leaking secrets into logs.

---

# Key DevOps principle

> **Make configuration explicit, versioned, validated, environment-specific, and independently managed from application code. Never depend on a developer's interactive shell for production behavior.**
