# SSH, Bastion Hosts and Remote Execution for DevOps

## Scope

This topic covers SSH from a **DevOps Engineer perspective**:

- Secure remote access to servers
- Public/private key authentication
- Bastion hosts and private subnets
- SSH troubleshooting
- Remote command execution
- File transfer with `scp`, `sftp`, and `rsync`
- Jenkins and Ansible SSH usage
- SSH hardening and operational security
- SSH versus AWS Systems Manager Session Manager

## 1. What is SSH?

SSH stands for **Secure Shell**. It provides encrypted communication between a client and a remote server.

Common DevOps uses:

- Connecting to Linux EC2 instances
- Accessing private servers through a bastion host
- Running deployment commands
- Copying configuration files
- Executing Ansible tasks
- Allowing Jenkins to deploy applications
- Troubleshooting services remotely

```bash
ssh username@server-ip
ssh -p 2222 ubuntu@10.0.2.15
```

The default SSH TCP port is `22`. This is a convention, not a requirement.

## 2. SSH Architecture

```text
SSH Client  ───── encrypted TCP connection ─────>  SSH Server
ssh command                                      sshd
```

| Component | Role |
|---|---|
| `ssh` | SSH client |
| `sshd` | SSH server daemon |
| `ssh-keygen` | Generates SSH keys |
| `ssh-agent` | Holds decrypted private keys in memory |
| `ssh-add` | Adds keys to an agent |
| `ssh-keyscan` | Collects public host keys |
| `scp` | Secure file copy |
| `sftp` | Interactive secure file transfer |
| `rsync` | Efficient file synchronization over SSH |

```bash
ssh -V
sudo systemctl status ssh
sudo ss -tulpn | grep ':22'
```

## 3. What Happens During an SSH Connection?

1. TCP connection is established.
2. Client and server exchange protocol versions.
3. Cryptographic algorithms are negotiated.
4. The server presents its host key.
5. The client verifies the host key against `known_hosts`.
6. Key exchange creates session encryption keys.
7. The client authenticates the user.
8. A shell or command session is opened.

Important distinction:

- **Host key** identifies the server.
- **User key** authenticates the user.

Host keys are stored on the client in:

```text
~/.ssh/known_hosts
```

User public keys are installed on the server in:

```text
~/.ssh/authorized_keys
```

## 4. Public-Key Authentication

SSH key authentication uses:

```text
Private key → kept secret on client
Public key  → copied to server
```

Generate an Ed25519 key:

```bash
ssh-keygen -t ed25519 -C "devops-user"
```

Generate RSA when required by legacy systems:

```bash
ssh-keygen -t rsa -b 4096 -C "devops-user"
```

Secure permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/known_hosts
```

Never copy the private key to a server.

## 5. Installing a Public Key

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@10.0.2.15
```

Manual method on the server:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chown -R ubuntu:ubuntu ~/.ssh
```

The public key must be one complete line in `authorized_keys`.

## 6. SSH Client Configuration

File:

```text
~/.ssh/config
```

Example:

```sshconfig
Host app-server
    HostName 10.0.2.15
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    ConnectTimeout 10
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

Connect:

```bash
ssh app-server
```

Protect the file:

```bash
chmod 600 ~/.ssh/config
```

Example environment aliases:

```sshconfig
Host dev-app
    HostName 10.0.2.15
    User ubuntu
    IdentityFile ~/.ssh/dev_ed25519

Host prod-app
    HostName 10.0.5.20
    User deploy
    IdentityFile ~/.ssh/prod_ed25519
```

## 7. Bastion Host / Jump Host

A bastion host is a controlled server used to access systems that are not directly reachable from the internet.

```text
Developer Laptop
       |
       v
Bastion Host in Public Subnet
       |
       v
Application EC2 in Private Subnet
```

Recommended security-group model:

```text
Bastion SG:
  TCP/22 only from trusted administrator IPs

Private App SG:
  TCP/22 only from Bastion SG
```

Avoid:

```text
TCP/22 from 0.0.0.0/0
```

A bastion is a network access point, not a replacement for authentication and authorization.

## 8. ProxyJump

Modern SSH uses `ProxyJump`:

```bash
ssh -J ubuntu@203.0.113.10 ubuntu@10.0.2.15
```

SSH configuration:

```sshconfig
Host bastion
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/bastion_ed25519

Host private-app
    HostName 10.0.2.15
    User ubuntu
    IdentityFile ~/.ssh/app_ed25519
    ProxyJump bastion
```

Connect:

```bash
ssh private-app
```

The bastion provides the network path. The target server still authenticates the user separately.

## 9. ProxyCommand

Legacy equivalent:

```bash
ssh   -o ProxyCommand="ssh -W %h:%p ubuntu@203.0.113.10"   ubuntu@10.0.2.15
```

Prefer `ProxyJump` unless a legacy environment requires `ProxyCommand`.

## 10. Bastion Network Requirements

Laptop to bastion:

- Bastion has a reachable address.
- Bastion security group allows TCP/22 from your IP.
- Routes and NACLs are correct.
- Bastion SSH service is running.

Bastion to private server:

- Bastion can route to the private subnet.
- Target security group allows TCP/22 from the bastion security group.
- NACLs allow forward and return traffic.
- Target is running.
- `sshd` is listening.
- Correct username and key are used.

A successful laptop-to-bastion connection does not prove bastion-to-target connectivity.

## 11. SSH Errors

### Connection timed out

Usually indicates routing or filtering problems.

```bash
nc -vz 10.0.2.15 22
ip route get 10.0.2.15
```

Possible causes:

- Wrong IP
- Missing route
- Security group or NACL block
- Private subnet is unreachable
- Instance is stopped
- SSH port is filtered

### Connection refused

The host is reachable, but no service is accepting the connection.

```bash
sudo systemctl status ssh
sudo ss -tulpn | grep ':22'
sudo journalctl -u ssh -n 100 --no-pager
```

### Permission denied (publickey)

The network works, but authentication failed.

```bash
ssh -vvv -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes ubuntu@10.0.2.15
```

Check:

- Username
- Private key
- `authorized_keys`
- File ownership
- File permissions
- `sshd_config`

### Host identification changed

Inspect:

```bash
ssh-keygen -F 10.0.2.15
```

After independently verifying the new server identity:

```bash
ssh-keygen -R 10.0.2.15
```

Do not blindly disable host-key verification.

## 12. Verbose Debugging

```bash
ssh -v user@host
ssh -vv user@host
ssh -vvv user@host
ssh -G private-app
```

`-vvv` helps identify:

- Loaded configuration
- Selected identity files
- ProxyJump behavior
- Host-key verification
- Authentication failure point

## 13. Remote Command Execution

```bash
ssh ubuntu@10.0.2.15 'hostname'
```

Multiple commands:

```bash
ssh ubuntu@10.0.2.15 '
  set -e
  hostname
  uptime
  systemctl is-active nginx
'
```

Run a local script remotely:

```bash
ssh host 'bash -s' < deploy.sh
```

Shell quoting matters:

```bash
ssh host "echo $HOME"
ssh host 'echo $HOME'
```

Double quotes expand `$HOME` locally. Single quotes allow the remote shell to expand it.

## 14. TTY and Non-Interactive Execution

Force a terminal:

```bash
ssh -t host 'sudo -i'
```

Use `-tt` only when required:

```bash
ssh -tt host 'sudo command'
```

For Jenkins and Ansible, prefer non-interactive commands. Unwanted TTY or password prompts can cause pipelines to hang.

## 15. Long-Running Commands

A normal SSH session may terminate when the connection closes.

```bash
ssh host 'nohup /opt/app/start.sh > /var/log/app.log 2>&1 &'
```

For production services, use `systemd` rather than relying on `nohup`.

For interactive work:

```bash
tmux new -s deployment
tmux attach -t deployment
```

Detach with:

```text
Ctrl+b, then d
```

## 16. SCP

Upload:

```bash
scp app.conf ubuntu@10.0.2.15:/tmp/
```

Download:

```bash
scp ubuntu@10.0.2.15:/var/log/app.log .
```

Copy a directory:

```bash
scp -r ./release ubuntu@10.0.2.15:/opt/
```

Through a bastion:

```bash
scp -o ProxyJump=bastion ./app.conf private-app:/tmp/
```

## 17. SFTP

```bash
sftp ubuntu@10.0.2.15
```

Common commands:

```text
pwd       remote directory
lpwd      local directory
ls        list remote files
lls       list local files
cd /opt   change remote directory
lcd /tmp  change local directory
put file  upload
get file  download
bye       exit
```

## 18. Rsync over SSH

```bash
rsync -avz ./release/ ubuntu@10.0.2.15:/opt/app/
```

Dry run:

```bash
rsync -avzn ./release/ host:/opt/app/
```

Delete remote files missing locally:

```bash
rsync -avz --delete ./release/ host:/opt/app/
```

Use `--delete` carefully.

Through a bastion:

```bash
rsync -avz -e "ssh -J bastion" ./release/ private-app:/opt/app/
```

Trailing slash matters:

```bash
rsync -av ./release/ host:/opt/app/
```

copies the contents, while:

```bash
rsync -av ./release host:/opt/app/
```

copies the directory itself.

## 19. SSH Agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-add -D
```

Agent forwarding:

```bash
ssh -A bastion
```

Security warning: a compromised bastion may abuse a forwarded agent. Prefer `ProxyJump` when possible and avoid global agent forwarding.

## 20. Connection Multiplexing

Repeated SSH connections can be slow. Configure:

```sshconfig
Host private-app
    HostName 10.0.2.15
    User ubuntu
    ControlMaster auto
    ControlPath ~/.ssh/control-%C
    ControlPersist 5m
```

Check:

```bash
ssh -O check private-app
```

Stop:

```bash
ssh -O exit private-app
```

This can improve Ansible and repeated deployment commands.

## 21. Timeouts and Keepalives

```bash
ssh -o ConnectTimeout=10 host
```

```bash
ssh   -o ServerAliveInterval=30   -o ServerAliveCountMax=3   host
```

Difference:

- `ConnectTimeout`: time allowed to establish the connection.
- `ServerAliveInterval`: keepalive interval after connection.
- `ServerAliveCountMax`: unanswered keepalive limit.

## 22. SSH Server Configuration

Main files:

```text
/etc/ssh/sshd_config
/etc/ssh/sshd_config.d/
```

Validate before applying:

```bash
sudo sshd -t
```

Reload:

```bash
sudo systemctl reload ssh
```

Useful hardening settings:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
AllowUsers ubuntu deploy
X11Forwarding no
```

Keep an existing administrative session open while changing SSH configuration. Do not disable password authentication until key access is confirmed.

## 23. SSH Hardening

- Disable direct root login.
- Prefer public-key authentication.
- Disable password authentication after validation.
- Restrict users with `AllowUsers` or `AllowGroups`.
- Use least-privilege deployment accounts.
- Protect private keys with passphrases.
- Restrict security-group access.
- Monitor SSH logs.
- Keep OpenSSH patched.
- Use `fail2ban` where appropriate.
- Verify host keys.
- Avoid `StrictHostKeyChecking=no` in production.

Logs:

```bash
sudo journalctl -u ssh
sudo tail -f /var/log/auth.log
```

## 24. Jenkins Considerations

Use Jenkins Credentials rather than hardcoding keys.

Best practices:

- Dedicated deployment user
- No private keys in Git
- No secrets printed in logs
- Non-interactive commands
- Restricted `sudoers` permissions
- Secure host-key management

Example:

```groovy
sshagent(credentials: ['prod-deploy-key']) {
    sh '''
        ssh -o BatchMode=yes deploy@server           'sudo systemctl restart myapp'
    '''
}
```

`BatchMode=yes` prevents interactive prompts and makes automation fail fast.

## 25. Ansible Considerations

Inventory example:

```ini
[app]
10.0.2.15 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Through a bastion:

```ini
[app]
private-app ansible_host=10.0.2.15 ansible_user=ubuntu

[all:vars]
ansible_ssh_common_args='-o ProxyJump=bastion'
```

Test:

```bash
ansible all -i inventory.ini -m ping
ansible app -m ping -vvvv
```

Check user, key, inventory hostname, proxy configuration, and key permissions when manual SSH works but Ansible fails.

## 26. SSH versus AWS SSM Session Manager

| Feature | SSH | SSM Session Manager |
|---|---|---|
| Network | Reachable TCP port | SSM agent and AWS API connectivity |
| Port 22 | Usually required | Not required |
| Bastion | Often required | Usually not required |
| Authentication | SSH keys/passwords | IAM permissions |
| Private key | Required for key auth | Not required |
| Audit | Additional setup | AWS logging integrations |
| Best fit | Traditional SSH tooling | Cloud-native EC2 operations |

SSM can avoid public IPs, bastion hosts, inbound port 22, and SSH key distribution.

Requirements include:

- SSM Agent
- Instance IAM role
- Network access to SSM endpoints
- Correct IAM permissions

```bash
aws ssm start-session --target i-xxxxxxxxxxxxxxxxx
```

## 27. Real-World Scenarios

### Scenario: Bastion works, private EC2 times out

Check:

1. Bastion route to private subnet.
2. Target SG allows SSH from bastion SG.
3. NACL forward and return traffic.
4. Target instance state.
5. `sshd` listener.
6. Correct private IP.

From bastion:

```bash
nc -vz 10.0.2.15 22
```

### Scenario: Jenkins deployment hangs

Likely causes:

- Host-key confirmation
- Key passphrase prompt
- Sudo password prompt
- Missing network route
- Remote command waiting for input

Use:

```bash
ssh -o BatchMode=yes -o ConnectTimeout=10 host 'command'
```

### Scenario: Manual SSH works but Ansible fails

Compare:

```bash
ssh app-server
ansible-inventory --graph
ansible app-server -m ping -vvvv
```

Check the effective user, key, inventory, proxy, and environment.

## 28. Interview Questions

1. **What is the difference between a host key and a user key?**  
   A host key identifies the server; a user key authenticates the user.

2. **Why should private keys never be copied to servers?**  
   Anyone possessing the key can impersonate its owner.

3. **What is a bastion host?**  
   A controlled jump server used to reach private systems.

4. **What is ProxyJump?**  
   An SSH mechanism that routes a connection through a jump host.

5. **What is the difference between timeout and refused?**  
   Timeout usually indicates filtering or routing problems. Refused means the host responded but no service accepted the port.

6. **How do you debug SSH authentication?**  
   Use `ssh -vvv`, then inspect username, key, `authorized_keys`, permissions, ownership, and server configuration.

7. **Why can SSH work manually but fail in Jenkins?**  
   Jenkins may use a different user, home directory, key, known-hosts file, network path, or non-interactive environment.

8. **Why is SSM useful for EC2?**  
   It can provide IAM-controlled access without inbound SSH or a bastion.

## 29. Command Reference

```bash
ssh host
ssh user@host
ssh -p 2222 user@host
ssh -i key.pem user@host
ssh -vvv user@host
ssh -J bastion user@private-host
ssh -o ConnectTimeout=10 host
ssh -o BatchMode=yes host 'command'
ssh -t host 'sudo -i'
ssh -G host
ssh-keygen -t ed25519
ssh-copy-id user@host
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-keygen -F host
ssh-keygen -R host
scp file host:/path/
sftp user@host
rsync -avz source/ host:/destination/
```

## 30. Final DevOps Checklist

- [ ] Correct route exists.
- [ ] Security groups allow only required sources.
- [ ] NACLs allow forward and return traffic.
- [ ] `sshd` is running.
- [ ] Correct username is used.
- [ ] Public key is installed correctly.
- [ ] Private-key permissions are secure.
- [ ] Host-key verification is enabled.
- [ ] Bastion access is restricted.
- [ ] Jenkins and Ansible credentials are secure.
- [ ] Automation is non-interactive.
- [ ] SSH logs are monitored.
- [ ] Root login is disabled.
- [ ] Password authentication is disabled where appropriate.
- [ ] SSM is considered for AWS EC2 access.

## Key DevOps Principle

> SSH is not only a login mechanism. In DevOps, it is a transport layer for deployment, configuration management, troubleshooting, and automation. Secure the network path, identity, credentials, host verification, and automation behavior together.
