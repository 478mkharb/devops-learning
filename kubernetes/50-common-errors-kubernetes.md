# Kubernetes Common Errors – Deep Explanation with Examples

This document explains **10 very common Kubernetes errors**, *why they happen*, *how to identify them*, and *how to fix them*, with **practical examples**.

---

## 1. ImagePullBackOff

### What it means

Kubernetes is **unable to pull the container image** from the registry.

### Common causes

* Wrong image name or tag
* Image does not exist
* Private registry without credentials
* Registry rate limiting (Docker Hub)

### How to identify

```bash
kubectl describe pod <pod-name>
```

You’ll see:

```
Failed to pull image "nginx:latestt"
```

### Example

```yaml
containers:
- name: web
  image: nginx:latestt   # typo
```

### Fix

* Correct image name/tag
* Add imagePullSecrets for private registry

```bash
kubectl create secret docker-registry regcred ...
```

---

## 2. CrashLoopBackOff

### What it means

Container **starts → crashes → restarts repeatedly**.

### Common causes

* Application crashes on startup
* Wrong command / entrypoint
* Missing environment variables
* App exits immediately

### How to identify

```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

### Example

```yaml
command: ["/bin/bash", "-c", "exit 1"]
```

### Fix

* Check logs
* Fix startup command
* Ensure app runs in foreground

---

## 3. NodeNotReady

### What it means

Node **cannot communicate with control plane**.

### Common causes

* kubelet stopped
* Disk pressure / memory pressure
* Network issues
* Node rebooted

### How to identify

```bash
kubectl get nodes
kubectl describe node <node>
```

### Fix

* Restart kubelet
* Free disk space
* Check node networking

---

## 4. PersistentVolumeClaim (PVC) Pending

### What it means

PVC exists but **no matching PV found**.

### Common causes

* No available PersistentVolume
* StorageClass misconfigured
* Wrong access mode

### Example

```yaml
resources:
  requests:
    storage: 20Gi
```

But only 10Gi PV exists.

### Fix

* Create matching PV
* Verify StorageClass

```bash
kubectl get sc
```

---

## 5. Pod Stuck in Pending State

### What it means

Pod **cannot be scheduled** on any node.

### Common causes

* Insufficient CPU/memory
* NodeSelector / Taints
* PVC pending

### How to identify

```bash
kubectl describe pod <pod>
```

### Fix

* Reduce resource requests
* Add more nodes
* Fix taints/tolerations

---

## 6. RBAC Access Denied

### What it means

User or service account **does not have permission**.

### Error example

```
Error from server (Forbidden)
```

### Common causes

* Missing RoleBinding
* Wrong ServiceAccount

### Fix

```bash
kubectl create rolebinding ...
```

Always check:

```bash
kubectl auth can-i get pods --as=system:serviceaccount:ns:sa
```

---

## 7. Service Unreachable

### What it means

Service exists but **traffic doesn’t reach pods**.

### Common causes

* Wrong selector labels
* Pod not listening on targetPort
* NetworkPolicy blocking traffic

### Example

```yaml
selector:
  app: backend
```

But pod label is `app: api`.

### Fix

* Match labels
* Verify ports
* Test with port-forward

---

## 8. ResourceQuota Exceeded

### What it means

Namespace **hit CPU/memory/pod limits**.

### Error example

```
exceeded quota
```

### Fix

* Increase quota
* Delete unused resources

```bash
kubectl get resourcequota
```

---

## 9. Evicted Pods

### What it means

Pod **killed by kubelet** due to node pressure.

### Common causes

* Memory pressure
* Disk pressure

### How to identify

```bash
kubectl describe pod <pod>
```

Status:

```
Reason: Evicted
```

### Fix

* Increase node resources
* Set resource limits

---

## 10. Deployment Rollout Fails

### What it means

New version **never becomes ready**.

### Common causes

* Readiness probe failing
* CrashLoopBackOff in new pods
* Image pull errors

### How to identify

```bash
kubectl rollout status deployment <name>
```

### Fix

* Check probes
* Rollback if needed

```bash
kubectl rollout undo deployment <name>
```

---

## Final Tip

When debugging Kubernetes issues, always follow this order:

1. `kubectl get`
2. `kubectl describe`
3. `kubectl logs`

This alone solves **80% of real-world issues**.
