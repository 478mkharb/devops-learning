# DevOps L2 Mock Interview – Revision Notes

## 1. Linux Troubleshooting – Server Slow Scenario

### Interview Question

A production server suddenly becomes slow and API response time increases significantly. What troubleshooting steps would you take?

### Structured Troubleshooting Approach

#### Step 1 – Check System Load

Command:

```
uptime
```

or

```
top
```

Reason:
Identify system load and compare with number of CPU cores.

---

#### Step 2 – Identify CPU-consuming Processes

Command:

```
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
```

or

```
top
```

Reason:
Find processes consuming excessive CPU.

---

#### Step 3 – Check Memory Usage

Command:

```
free -m
```

Check for OOM events:

```
dmesg | grep -i oom
```

---

#### Step 4 – Check Disk Usage

Command:

```
df -h
```

Reason:
If disk reaches 100%, applications become slow or fail.

---

#### Step 5 – Check Inode Usage

Command:

```
df -i
```

Reason:
If inodes are exhausted, new files cannot be created even if disk space exists.

---

#### Step 6 – Check Disk I/O

Command:

```
iostat -x 1
```

or

```
iotop
```

High I/O wait indicates disk bottleneck.

---

#### Step 7 – Check Network

Command:

```
ss -tulpn
```

Check for abnormal number of connections.

---

#### Step 8 – Check Logs

Command:

```
journalctl -xe
```

Also inspect:

```
/var/log/syslog
/var/log/messages
```

---

## 2. Linux Load Average

### Interview Question

What does load average mean in Linux?

### Definition

Load average represents the number of processes that are:

* Running on CPU
* Waiting for CPU
* Waiting for uninterruptible I/O

States counted:

* R (Running)
* D (Uninterruptible sleep – usually disk I/O)

---

### Load Average Interpretation

Example:

```
load average: 12.15, 11.90, 10.50
```

| Value | Meaning         |
| ----- | --------------- |
| 12.15 | Last 1 minute   |
| 11.90 | Last 5 minutes  |
| 10.50 | Last 15 minutes |

---

### CPU Core Rule

Healthy load ≈ Number of CPU cores

Example:

4 CPU cores:

| Load | Meaning          |
| ---- | ---------------- |
| 1    | One CPU utilized |
| 4    | Full utilization |
| 8    | Overloaded       |
| 12   | Severe overload  |

---

## 3. Kubernetes CrashLoopBackOff

### Interview Question

What does CrashLoopBackOff mean and how do you troubleshoot it?

### Meaning

CrashLoopBackOff occurs when:

Container starts → crashes → Kubernetes restarts it → crashes again.

Kubernetes applies exponential restart delay:

10s → 20s → 40s → 80s.

---

### Troubleshooting Steps

#### Step 1 – Check Pod Status

```
kubectl get pods -n <namespace>
```

---

#### Step 2 – Describe Pod

```
kubectl describe pod <podname> -n <namespace>
```

Look for:

* Events
* Restart count
* Probe failures

---

#### Step 3 – Check Logs

```
kubectl logs <podname>
```

If container restarted:

```
kubectl logs <podname> --previous
```

---

#### Step 4 – Check Exit Codes

Example:

Exit Code 137 → OOMKilled

---

### Common Causes

| Cause                          | Explanation                           |
| ------------------------------ | ------------------------------------- |
| Application crash              | Code error                            |
| OOMKilled                      | Memory limit exceeded                 |
| Liveness probe failure         | Kubernetes keeps restarting container |
| Missing environment variables  | App fails during startup              |
| Missing ConfigMap/Secret       | Configuration missing                 |
| Dependency service unavailable | Cannot connect to DB or API           |

---

## 4. Kubernetes Cross-Namespace Communication

### Interview Question

How do pods communicate across namespaces?

### Kubernetes Service DNS

Format:

```
service-name.namespace.svc.cluster.local
```

Example:

```
orders-service.orders.svc.cluster.local
```

Short DNS also works:

```
orders-service.orders
```

---

### Troubleshooting Steps

#### Step 1 – Verify Service

```
kubectl get svc -n orders
```

---

#### Step 2 – Check Endpoints

```
kubectl get endpoints orders-service -n orders
```

If endpoints are empty, service is not linked to pods.

---

#### Step 3 – Check Labels

Service uses label selectors.

Check service selector:

```
kubectl describe svc orders-service
```

Check pod labels:

```
kubectl get pods --show-labels
```

---

#### Step 4 – Test DNS Inside Pod

```
kubectl exec -it payments-pod -n payments -- sh
```

Test DNS:

```
nslookup orders-service.orders
```

Test connectivity:

```
curl orders-service.orders:8080
```

---

#### Step 5 – Check Network Policies

```
kubectl get networkpolicy -A
```

Network policies can block traffic.

---

## 5. CI/CD Failure – Docker Build Error

### Interview Question

Jenkins pipeline fails with:

```
no space left on device
```

But server shows 200GB free disk.

---

### Possible Reasons

1. Docker storage full
2. Inodes exhausted
3. Overlay filesystem full
4. /tmp partition full
5. Docker layer cache full

---

### Troubleshooting Steps

#### Step 1 – Check Disk Usage

```
df -h
```

---

#### Step 2 – Check Inodes

```
df -i
```

---

#### Step 3 – Check Docker Storage

```
docker system df
```

---

#### Step 4 – Check Docker Directory

```
du -sh /var/lib/docker
```

---

### Cleanup Commands

Remove unused Docker resources:

```
docker system prune -a
```

Remove dangling images:

```
docker image prune
```

---

### Jenkins Maintenance

Old builds accumulate under:

```
/var/lib/jenkins/jobs
```

Enable Jenkins feature:

"Discard Old Builds"

---

### Best Practice Architecture

Jenkins Master should only orchestrate builds.

Architecture:

Jenkins Master
↓
Jenkins Agents
↓
Docker Builds

Benefits:

* scalable
* isolated builds
* prevents master overload

---
### 6. Jenkins Cannot Access Kubernetes Cluster 

### Interview Question

Jenkins pipeline deployment to Kubernetes is failing with a connection or authentication error. Jenkins cannot access the Kubernetes cluster. How would you troubleshoot this issue?

---

### Answer (L2 Troubleshooting Approach)

This issue typically occurs when Jenkins cannot authenticate or communicate with the Kubernetes API server. A structured troubleshooting process is expected in DevOps L2 interviews.

---

### Step 1 — Verify kubeconfig configuration

Jenkins needs a **kubeconfig file** to know which cluster to connect to and which credentials to use.

Test from the Jenkins server or agent:

kubectl get nodes

If this fails, Jenkins does not have proper cluster access.

Common problems:

* kubeconfig file missing
* wrong API server endpoint
* incorrect credentials

---

### Step 2 — Check authentication credentials

Kubernetes authentication can be done using:

* Service account token
* Client certificates
* Cloud IAM roles (EKS/GKE/AKS)

Common error:

"You must be logged in to the server"

Fix:

* Verify token in kubeconfig
* Update credentials stored in Jenkins

---

### Step 3 — Verify RBAC permissions

Sometimes Jenkins can connect to the cluster but does not have permission to create or modify resources.

Example error:

User "jenkins" cannot create deployments

Check permissions:

kubectl auth can-i create deployment

Fix:
Create RoleBinding or ClusterRoleBinding to give Jenkins the required permissions.

---

### Step 4 — Check network connectivity to Kubernetes API server

Jenkins must reach the Kubernetes API server (usually port **6443**).

Test connectivity:

curl https://<k8s-api-server>:6443

Possible causes:

* firewall blocking traffic
* security group restriction
* private cluster endpoint

---

### Step 5 — Verify kubectl availability on Jenkins agent

If the pipeline runs on an agent container, kubectl might not be installed.

Example error:

kubectl: command not found

Fix:

* Install kubectl on Jenkins node
* Use a Docker image containing kubectl

---

### Step 6 — Check Jenkins Kubernetes plugin configuration

If Jenkins dynamically creates agents using Kubernetes plugin:

Manage Jenkins → Configure System → Kubernetes Cloud

Verify:

* Kubernetes API URL
* credentials
* namespace

---

### Step 7 — Verify correct cluster context

Sometimes kubeconfig contains multiple clusters and the wrong context is selected.

Check:

kubectl config current-context

Switch if necessary.

---

### Step 8 — Check Jenkins logs

Logs often show authentication or connectivity errors.

Typical location:

/var/log/jenkins/jenkins.log

---

## Kubernetes Pod Stuck in Pending

### Problem

Deployment has 3 replicas but one pod shows **Pending**.

### Troubleshooting Steps

1. Check pod status

```
kubectl get pods -o wide
```

2. Describe the pod (most important)

```
kubectl describe pod <pod-name>
```

Look at **Events** section.

Possible messages:

* insufficient CPU
* node selector mismatch
* PVC not bound

3. Check cluster nodes

```
kubectl get nodes
kubectl describe node <node-name>
```

4. Check ResourceQuota

```
kubectl get resourcequota -n <namespace>
```

5. Check PVC

```
kubectl get pvc
kubectl describe pvc <pvc-name>
```

### Example Root Causes

* insufficient CPU/memory
* node selector mismatch
* storage unavailable

---

## Kubernetes ImagePullBackOff

### Problem

Pod fails with **ImagePullBackOff**.

### Troubleshooting Steps

1. Check pods

```
kubectl get pods
```

2. Describe pod

```
kubectl describe pod <pod-name>
```

3. Verify image exists
   Check container registry.

4. Check imagePullSecrets

```
kubectl get secrets
```

### Example Fix

Build and push image:

```
docker build -t myapp:v1 .
docker push myrepo/myapp:v1
```

---

## Linux Server Slow

### Steps

1. Check load

```
uptime
```

2. Check CPU

```
top
```

3. Check memory

```
free -h
```

4. Check processes

```
ps aux --sort=-%cpu | head
```

5. Check disk

```
df -h
```

6. Check IO

```
iostat -x 1
```

### Example Causes

* memory leak
* high CPU process
* disk full

---

## Git Commit Pushed to Wrong Branch

### Scenario

Commit pushed to **main instead of dev**.

### Steps

1. Find commit

```
git log
```

2. Switch to dev

```
git checkout dev
```

3. Cherry pick commit

```
git cherry-pick <commit-id>
```

4. Push branch

```
git push origin dev
```

5. Remove from main

```
git revert <commit-id>
```

---

## AWS EC2 Website Not Reachable

### Check AWS Networking

1. Security Group

* allow port 80/443

2. Network ACL

3. Route table

```
0.0.0.0/0 → Internet Gateway
```

### Check Server

Check service

```
systemctl status nginx
```

Check port

```
ss -tulpn
```

Check logs

```
tail -f /var/log/nginx/error.log
```

---

## Terraform Resource Already Exists

### Solution: Import resource

1. Write resource block

```
resource "aws_instance" "web" {
}
```

2. Import resource

```
terraform import aws_instance.web i-123456789
```

3. Verify state

```
terraform state list
```

4. Run plan

```
terraform plan
```

---

## Docker Container Exits Immediately

### Steps

1. Check container

```
docker ps -a
```

2. Check logs

```
docker logs <container-id>
```

3. Inspect container

```
docker inspect <container-id>
```

4. Run interactive shell

```
docker run -it <image> /bin/bash
```

### Common Reasons

* application crash
* wrong CMD
* missing environment variables

---

## Ansible SSH UNREACHABLE

### Steps

1. Test ansible connectivity

```
ansible all -m ping
```

2. Verify inventory

```
ansible-inventory --list
```

3. Test manual SSH

```
ssh user@host
```

4. Configure key

```
ssh-keygen
ssh-copy-id user@host
```

### Possible Causes

* SSH key missing
* wrong user
* firewall blocking port 22

---

## Jenkins OutOfMemoryError

### Steps

Check logs

```
journalctl -u jenkins
```

Check system memory

```
free -h
```

Increase heap size

Edit:

```
/etc/default/jenkins
```

Example

```
JENKINS_JAVA_OPTIONS="-Xms2g -Xmx4g"
```

Restart Jenkins

```
systemctl restart jenkins
```

---

## Pod Running But Application Not Accessible

### Steps

1. Check pods

```
kubectl get pods
```

2. Check service

```
kubectl get svc
```

3. Check endpoints

```
kubectl get endpoints <service>
```

4. Verify labels

```
kubectl get pods --show-labels
```

5. Check logs

```
kubectl logs <pod>
```

### Example Issues

* label mismatch
* wrong targetPort
* readiness probe failing

---

# Maven Question

## Scenario

Jenkins Maven build fails with:

```
Could not resolve dependencies for project
```

## Troubleshooting Steps

### Step 1: Check Jenkins build logs

```
mvn clean install
```

Look for the exact dependency error.

### Step 2: Check pom.xml

Example dependency:

```
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-core</artifactId>
  <version>5.2.0</version>
</dependency>
```

Verify:

* dependency version exists
* groupId and artifactId correct

### Step 3: Check Maven repository connectivity

```
curl https://repo.maven.apache.org
```

### Step 4: Clear corrupted Maven cache

```
rm -rf ~/.m2/repository
```

### Root Causes

* dependency version removed
* corrupted .m2 cache
* repository connectivity issue

---

# Maven Question

## Scenario

Maven build time increased from 5 minutes to 20 minutes.

## Troubleshooting

### Check Maven cache

```
ls ~/.m2/repository
```

If deleted, dependencies are re-downloaded.

### Check if tests are running

```
mvn clean install -DskipTests
```

### Enable parallel builds

```
mvn clean install -T 1C
```

### Check Jenkins agent resources

```
top
free -h
```

### Possible Causes

* missing .m2 cache
* tests executing
* low CPU/memory

---

# CI/CD Question

## Scenario

Jenkins pipeline fails with:

```
Cannot connect to the Docker daemon
```

## Troubleshooting

### Check Docker installation

```
docker --version
```

### Check Docker service

```
systemctl status docker
```

Start Docker

```
systemctl start docker
```

### Check Jenkins permissions

```
groups jenkins
```

Add Jenkins to docker group

```
usermod -aG docker jenkins
systemctl restart jenkins
```

### Check docker socket

```
ls -l /var/run/docker.sock
```

---

# CI/CD Question

## Scenario

Jenkins pipeline succeeds but new application version is not deployed to Kubernetes.

## Troubleshooting

### Check Jenkins deployment stage

Verify pipeline step:

```
kubectl apply -f deployment.yaml
```

### Check Kubernetes deployment

```
kubectl get deployments
```

### Check rollout status

```
kubectl rollout status deployment/app
```

### Verify image version

```
kubectl describe deployment app
```

Possible Issue:

* same image tag used
* rollout not triggered

Fix:

```
kubectl rollout restart deployment app
```

---

# Ansible Question

## Scenario

Playbook fails with:

```
UNREACHABLE! Failed to connect via ssh
```

## Troubleshooting

### Test Ansible connectivity

```
ansible all -m ping
```

### Verify inventory

```
ansible-inventory --list
```

### Test SSH manually

```
ssh user@host
```

### Configure SSH key

```
ssh-keygen
ssh-copy-id user@host
```

### Root Causes

* wrong SSH key
* wrong username
* port 22 blocked

---

# Ansible Question

## Scenario

Ansible playbook runs successfully but configuration is not applied.

## Troubleshooting

Check playbook execution

```
ansible-playbook playbook.yml -vvv
```

Check task results.

Verify module execution.

Example:

```
ansible webservers -m shell -a "nginx -v"
```

Check idempotency issues.

---


# Linux Question

## Scenario

Disk space suddenly becomes full.

## Troubleshooting

Check filesystem usage

```
df -h
```

Find large directories

```
du -sh /*
```

Check log files

```
/var/log
```

Clear logs

```
truncate -s 0 /var/log/app.log
```

Use logrotate.

---
