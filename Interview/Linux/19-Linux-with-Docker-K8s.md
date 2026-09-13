# Linux with Docker, Kubernetes and Cloud — DevOps Interview Handbook

> **Focus:** Linux concepts required to operate containers, Kubernetes workloads, and cloud infrastructure from a DevOps perspective.

---

## 1. Why Linux Matters in Modern DevOps

Even when teams use Docker, Kubernetes, and cloud services, Linux remains underneath the platform.

```text
Application
    |
    v
Container
    |
    v
Container runtime
    |
    v
Linux namespaces + cgroups
    |
    v
Linux kernel
    |
    v
VM / EC2 / Cloud infrastructure
```

Linux provides:

- Process isolation
- Resource control
- Networking
- Filesystems
- Security boundaries
- Device access
- Service management
- Logging
- Performance monitoring

A Kubernetes Pod is not a replacement for Linux knowledge. Kubernetes schedules workloads, while the Linux kernel executes and isolates them.

---

## 2. Linux Host Requirements for Containers

Inspect the host:

```bash
uname -a
cat /etc/os-release
nproc
free -h
df -h
lsmod
```

Important requirements include:

- Compatible Linux kernel
- Sufficient CPU and memory
- Available disk space
- Working networking
- Container runtime
- Correct cgroup configuration
- Required kernel modules
- Proper security policy

Check cgroups:

```bash
mount | grep cgroup
stat -fc %T /sys/fs/cgroup
```

Check namespaces:

```bash
ls -l /proc/1/ns
```

A container runtime depends on kernel features. If the host kernel or security policy is incompatible, containers may fail before the application starts.

---

## 3. Containers vs Virtual Machines

| Feature | Container | Virtual machine |
|---|---|---|
| Kernel | Shares host kernel | Has guest kernel |
| Startup | Usually fast | Usually slower |
| Isolation | Process-level | Hardware/VM-level |
| Resource overhead | Lower | Higher |
| Packaging | Application + dependencies | Full guest OS |
| Typical use | Microservices, CI jobs | Stronger isolation, legacy workloads |

A container is a group of processes isolated by Linux features. It is not a complete operating system.

---

## 4. Linux Namespaces

Namespaces isolate what a process can see.

Common namespaces:

| Namespace | Isolation |
|---|---|
| PID | Process IDs |
| NET | Network interfaces, routes, ports |
| MNT | Mount points |
| UTS | Hostname/domain name |
| IPC | Inter-process communication |
| USER | User/group IDs |
| CGROUP | Cgroup view |

Inspect a process:

```bash
ls -l /proc/<PID>/ns
```

Run a process in a new namespace:

```bash
sudo unshare --pid --fork --mount-proc bash
ps aux
```

The process may see a different PID hierarchy from the host.

### Important interview point

Namespaces provide visibility isolation. They do not alone guarantee complete security isolation.

---

## 5. Linux cgroups

Control groups limit and account for resource usage.

Common controls:

- CPU
- Memory
- PIDs
- Block I/O
- Devices

Inspect cgroup membership:

```bash
cat /proc/<PID>/cgroup
systemd-cgls
systemd-cgtop
```

Memory pressure indicators:

```bash
free -h
dmesg -T | grep -i oom
journalctl -k | grep -i oom
```

A container memory limit can cause the process to be killed even when the host still has some free memory.

### CPU vs memory behavior

- CPU limit usually causes throttling.
- Memory limit can cause OOM termination.
- PID limit can prevent new processes from starting.

---

## 6. Docker Linux Operations

Useful commands:

```bash
docker version
docker info
docker ps
docker ps -a
docker images
docker inspect <container>
docker logs <container>
docker stats
```

Check the Docker service:

```bash
sudo systemctl status docker
sudo journalctl -u docker -n 100 --no-pager
```

Check Docker disk usage:

```bash
docker system df
```

Clean unused objects carefully:

```bash
docker image prune
docker container prune
docker volume prune
docker system prune
```

Do not use destructive prune commands on production hosts without confirming what will be removed.

---

## 7. Docker Storage and Overlay Filesystems

Container filesystems are commonly layered.

Typical concepts:

```text
Read-only image layers
        +
Writable container layer
        =
Container filesystem
```

Inspect storage:

```bash
docker info | grep -i -E 'storage|driver'
df -h
du -sh /var/lib/docker
```

Common problems:

- Container writable layer grows
- JSON logs consume disk
- Unused images remain
- Volumes consume space
- Deleted files remain open

Find deleted-but-open files:

```bash
sudo lsof +L1
```

Persistent application data should normally use volumes or external storage rather than the writable container layer.

---

## 8. Docker Networking

Inspect networks:

```bash
docker network ls
docker network inspect bridge
```

Check container ports:

```bash
docker port <container>
ss -lntp
```

Common network modes:

- `bridge`
- `host`
- `none`
- Overlay networks in orchestrated environments

Example:

```bash
docker run -d --name web -p 8080:80 nginx
curl -I http://127.0.0.1:8080
```

### Common failure

The application listens on:

```text
127.0.0.1:8080
```

inside the container instead of:

```text
0.0.0.0:8080
```

A process bound to loopback may not be reachable through the container network.

---

## 9. Docker Security

Avoid running containers as root where possible.

Inspect:

```bash
docker inspect --format '{{.Config.User}}' <container>
```

Security controls include:

- Non-root user
- Read-only root filesystem
- Dropped Linux capabilities
- No privileged mode
- Restricted device access
- Resource limits
- Secret management
- Image scanning
- Minimal base image

Dangerous options:

```bash
--privileged
-v /:/host
-v /var/run/docker.sock:/var/run/docker.sock
```

These can provide very broad host access.

---

## 10. Kubernetes Node Architecture

```text
Kubernetes Control Plane
        |
        v
Scheduler/API/Controllers
        |
        v
Worker Node
  +---------------------------+
  | kubelet                   |
  | container runtime         |
  | kube-proxy / networking   |
  | Linux kernel              |
  | Pods                      |
  +---------------------------+
```

A worker node needs:

- kubelet
- Container runtime
- Network plugin
- Working DNS/networking
- Sufficient resources
- Correct time synchronization
- Required kernel settings

Inspect a node:

```bash
kubectl get nodes -o wide
kubectl describe node <node>
```

On the host:

```bash
systemctl status kubelet
journalctl -u kubelet -n 100 --no-pager
```

---

## 11. Kubernetes Pods and Linux Processes

A Pod is a logical group of containers sharing:

- Network namespace
- IP address
- Port space
- Optional volumes
- Lifecycle boundary

Inspect processes on a node:

```bash
ps -ef
crictl ps
crictl pods
```

A container crash may be caused by:

- Application exit
- Missing configuration
- Permission problem
- OOM kill
- Failed health check
- Missing mount
- Runtime failure

Useful commands:

```bash
kubectl get pod <pod> -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

---

## 12. Kubernetes Resource Requests and Limits

Requests influence scheduling:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

Limits cap usage:

```yaml
resources:
  limits:
    cpu: "1"
    memory: "512Mi"
```

### Linux interpretation

- CPU `250m` means one-quarter of a CPU core.
- Memory is enforced through cgroups.
- CPU overuse may be throttled.
- Memory overuse may trigger OOM behavior.

Inspect usage:

```bash
kubectl top nodes
kubectl top pods
```

On the node:

```bash
free -h
systemd-cgtop
```

---

## 13. Kubernetes Volumes and Linux Mounts

Common volume types:

- `emptyDir`
- `hostPath`
- PersistentVolume
- PersistentVolumeClaim
- Cloud block storage
- Network filesystems

Inspect mounts:

```bash
findmnt
mount
df -h
```

### `hostPath` warning

`hostPath` exposes a host filesystem path to a Pod and can create security and portability problems.

Use persistent storage abstractions where possible.

Common mount failures:

```text
MountVolume.SetUp failed
permission denied
read-only file system
no such file or directory
```

Troubleshoot:

```bash
kubectl describe pod <pod>
journalctl -u kubelet
df -h
findmnt
```

---

## 14. Kubernetes Networking from a Linux View

Important layers:

```text
Application process
    |
Pod network namespace
    |
CNI plugin
    |
Node network interface
    |
Cloud VPC/subnet/security rules
```

Inspect Pod IPs:

```bash
kubectl get pods -o wide
kubectl get svc
kubectl get endpoints
```

Test from a Pod:

```bash
kubectl exec -it <pod> -- sh
```

Inside the Pod:

```bash
ip addr
ip route
cat /etc/resolv.conf
getent hosts kubernetes.default
```

On the node:

```bash
ip addr
ip route
ss -lntp
```

Common causes of connectivity failure:

- Wrong Service selector
- Application bound to loopback
- NetworkPolicy
- CNI failure
- DNS failure
- Cloud security group
- Route table issue
- Wrong target port

---

## 15. Kubernetes DNS

Inspect:

```bash
cat /etc/resolv.conf
getent hosts kubernetes.default
nslookup service.namespace.svc.cluster.local
```

A Service normally provides stable service discovery while Pod IPs can change.

Common DNS failures:

- CoreDNS unavailable
- Network path broken
- Wrong namespace
- Wrong service name
- Search domain misunderstanding
- Upstream DNS unavailable

Check:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

## 16. Kubernetes Security Context

A security context controls how a Pod or container runs.

Example:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

Linux concepts involved:

- UID/GID
- Capabilities
- Seccomp
- AppArmor
- SELinux
- Filesystem permissions
- Privilege escalation

A container running as UID `10001` must have access to required files and directories.

Check file ownership inside an image or container:

```bash
id
ls -ln
stat /path/to/file
```

---

## 17. Linux on AWS EC2

Typical cloud layout:

```text
VPC
 |
 +--> Subnet
       |
       +--> EC2 instance
              |
              +--> Linux OS
                    |
                    +--> systemd
                    +--> application
                    +--> monitoring agent
                    +--> SSM Agent
```

Useful commands:

```bash
hostnamectl
uname -r
df -h
free -h
ip addr
ip route
ss -lntp
systemctl status amazon-ssm-agent
```

For private EC2 instances, verify:

- Route table
- NAT gateway or VPC endpoints
- Security groups
- Network ACLs
- IAM instance profile
- SSM Agent
- DNS resolution
- Time synchronization

---

## 18. SSM Instead of SSH

Systems Manager can provide shell access and automation without opening inbound SSH.

Check agent:

```bash
sudo systemctl status amazon-ssm-agent
sudo journalctl -u amazon-ssm-agent -n 100 --no-pager
```

Common SSM failures:

- Instance profile missing permissions
- Agent stopped
- No network path to SSM endpoints
- Incorrect region
- Instance not registered
- System clock issue
- Proxy misconfiguration

The Linux troubleshooting approach remains the same:

```bash
systemctl status amazon-ssm-agent
journalctl -u amazon-ssm-agent
getent hosts ssm.<region>.amazonaws.com
curl -I https://ssm.<region>.amazonaws.com
```

---

## 19. Cloud Block Storage and Filesystems

Cloud disks appear to Linux as block devices.

Inspect:

```bash
lsblk
blkid
findmnt
df -h
```

Typical process:

```text
Attach volume
   |
   v
Detect device
   |
   v
Partition if required
   |
   v
Create filesystem
   |
   v
Mount
   |
   v
Persist in /etc/fstab
```

Example:

```bash
sudo mkfs.ext4 /dev/xvdf
sudo mkdir -p /data
sudo mount /dev/xvdf /data
df -h /data
```

Do not format a device until you have confirmed its identity. Formatting destroys existing filesystem data.

---

## 20. Cloud Monitoring and Linux Metrics

Important metrics:

| Metric | Meaning |
|---|---|
| CPU utilization | CPU consumption |
| Load average | Runnable/uninterruptible work |
| Memory available | Memory headroom |
| Disk usage | Filesystem capacity |
| Inodes | Filesystem object capacity |
| Disk I/O wait | CPU waiting on I/O |
| Network throughput | Traffic volume |
| Process count | Process pressure |
| Open files | File descriptor usage |

Commands:

```bash
uptime
free -h
df -h
df -i
vmstat 1 5
iostat -xz 1 5
ss -s
ulimit -n
```

Cloud monitoring should be combined with Linux-level investigation. A high CPU alarm alone does not identify the responsible process.

---

## 21. Image Building with Packer

A Linux image pipeline commonly:

```text
Base image
   |
   v
Install packages
   |
   v
Copy configuration
   |
   v
Harden OS
   |
   v
Install monitoring/SSM
   |
   v
Validate services
   |
   v
Create image
```

Validate inside the build instance:

```bash
systemctl --failed
df -h
free -h
ss -lntp
journalctl -p warning -b
```

Golden image principles:

- Remove temporary credentials
- Remove build artifacts
- Avoid hardcoded secrets
- Use immutable versioning
- Patch before capture
- Verify startup services
- Confirm cloud-init behavior
- Confirm SSM registration
- Keep image provenance

---

## 22. Cloud-Init and First Boot

Cloud-init often configures a Linux cloud instance at first boot.

Inspect:

```bash
cloud-init status --long
journalctl -u cloud-init
journalctl -u cloud-final
ls -lah /var/log/cloud-init*
```

Common failures:

- YAML indentation
- Wrong package name
- Network unavailable
- User-data script not executable
- Command exits early
- Incorrect file ownership
- Service starts before configuration exists

Use explicit shell safety:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

Log important steps and validate the final state.

---

## 23. Linux Troubleshooting for Containers

### Container exits immediately

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

Check:

- Entrypoint
- Command
- Environment
- Working directory
- File permissions
- Required configuration
- Port binding

### Container cannot write

```bash
docker exec -it <container> sh
id
df -h
ls -ld /path
```

Check UID/GID and mounted volume ownership.

### Container cannot reach another service

```bash
docker exec <container> getent hosts service-name
docker network inspect <network>
ss -lntp
```

### Kubernetes Pod is OOMKilled

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState}'
kubectl top pod <pod>
```

Compare memory limits with application usage.

---

## 24. Linux Troubleshooting for Kubernetes

### `CrashLoopBackOff`

Investigate:

```bash
kubectl logs <pod> --previous
kubectl describe pod <pod>
kubectl get events --sort-by=.lastTimestamp
```

### `ImagePullBackOff`

Check:

- Image name/tag
- Registry DNS
- Registry credentials
- Network access
- Image architecture
- Pull secret

### `Pending`

Check:

```bash
kubectl describe pod <pod>
kubectl get nodes
kubectl get events
```

Possible causes:

- Insufficient resources
- Node taints
- Missing toleration
- Affinity rules
- Unbound PVC

### `Permission denied`

Check:

- Security context
- UID/GID
- Volume ownership
- SELinux/AppArmor
- Read-only filesystem
- Init container behavior

---

## 25. Example Container Health Checks

A process being alive does not mean the application is healthy.

Example Docker health check:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3   CMD curl --fail http://127.0.0.1:8080/health || exit 1
```

Example Kubernetes probe:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 20
```

- Readiness controls whether traffic is sent.
- Liveness determines whether the container should be restarted.
- Startup probes protect slow-starting applications.

---

## 26. Important Security Mistakes

Avoid:

```bash
docker run --privileged ...
```

Avoid mounting the host root:

```bash
-v /:/host
```

Avoid exposing the Docker socket unnecessarily:

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

Avoid running every container as root.

Avoid storing cloud credentials inside images.

Avoid embedding secrets in:

- Dockerfiles
- Image layers
- Kubernetes manifests
- Git repositories
- Shell history
- Logs
- Terraform state without protection

Use IAM roles, workload identity, secret managers, and short-lived credentials.

---

## 27. Interview Questions

### Q1. Is a container a virtual machine?

No. A container is an isolated group of processes sharing the host kernel. A VM includes a guest operating system and kernel.

### Q2. What do namespaces provide?

Namespaces isolate process visibility, networking, mounts, hostnames, IPC, users, and other kernel resources.

### Q3. What do cgroups provide?

Resource accounting and control for CPU, memory, PIDs, and I/O.

### Q4. Why does a container exit immediately?

Its main process exited, the command was incorrect, a required file/configuration was missing, or the process crashed.

### Q5. Why does a containerized service work inside the container but not from outside?

The service may bind to loopback, the port may not be published, a firewall/security group may block traffic, or the container network may be misconfigured.

### Q6. What is the difference between CPU and memory limits?

CPU overuse is generally throttled; memory overuse can cause OOM termination.

### Q7. Why is `hostPath` risky in Kubernetes?

It exposes host filesystem paths and reduces portability and isolation.

### Q8. How do you troubleshoot `CrashLoopBackOff`?

Inspect current and previous logs, Pod events, probes, environment variables, mounts, permissions, and resource limits.

### Q9. What is the difference between readiness and liveness?

Readiness controls traffic eligibility. Liveness controls restart decisions.

### Q10. Why can a Pod be Pending?

Insufficient resources, taints, affinity rules, missing tolerations, or storage binding problems.

### Q11. How can private EC2 instances be managed without SSH?

Use Systems Manager with correct IAM, SSM Agent, and network connectivity.

### Q12. Why should cloud credentials not be baked into images?

Images can be copied, inspected, and reused. Credentials may leak through image layers and logs.

---

## 28. Practical Command Checklist

### Host

```bash
uname -a
cat /etc/os-release
nproc
free -h
df -h
ip addr
ip route
ss -lntp
```

### Docker

```bash
docker info
docker ps -a
docker logs <container>
docker inspect <container>
docker stats
docker system df
```

### Kubernetes

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by=.lastTimestamp
kubectl top nodes
kubectl top pods
```

### Services

```bash
systemctl status docker
systemctl status kubelet
journalctl -u docker -n 100 --no-pager
journalctl -u kubelet -n 100 --no-pager
```

### Storage

```bash
lsblk
blkid
findmnt
df -h
df -i
```

### Cloud/SSM

```bash
systemctl status amazon-ssm-agent
journalctl -u amazon-ssm-agent -n 100 --no-pager
cloud-init status --long
```

---

## 29. Final DevOps Checklist

Before blaming Docker, Kubernetes, or the cloud platform, verify:

- [ ] Linux kernel and OS are supported
- [ ] CPU and memory are sufficient
- [ ] Disk blocks and inodes are available
- [ ] Required services are active
- [ ] Container runtime is healthy
- [ ] kubelet is healthy
- [ ] Network interfaces and routes are correct
- [ ] DNS resolution works
- [ ] Application binds to the correct interface
- [ ] Ports are published and allowed
- [ ] UID/GID permissions are correct
- [ ] Security context is compatible
- [ ] Volume mounts exist and are writable
- [ ] Resource requests and limits are appropriate
- [ ] Probes match application startup behavior
- [ ] Cloud routes and security rules are correct
- [ ] IAM/instance profile permissions are correct
- [ ] SSM Agent is online where required
- [ ] Logs and events have been inspected
- [ ] Images and artifacts are versioned and trusted
- [ ] Secrets are not embedded in images or manifests

---

## Key Takeaway

Docker, Kubernetes, and cloud platforms abstract Linux; they do not eliminate it.

When a workload fails, ask:

> **Which Linux process is running, under which UID, inside which namespace and cgroup, using which mount, network path, resource limit, and security policy?**

That question connects application behavior to the underlying infrastructure and resolves many container, Kubernetes, and cloud incidents.
