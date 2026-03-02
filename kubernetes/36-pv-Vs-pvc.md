# 📦 Difference Between PV and PVC (Kubernetes)

Persistent storage is a core concept in Kubernetes. **PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)** work together to provide durable storage to Pods, decoupling storage from application lifecycle.

---

## 🔹 What is a PersistentVolume (PV)?

A **PersistentVolume (PV)** is a **cluster-level storage resource** provisioned by:

* A **cluster administrator**, or
* Dynamically by Kubernetes using a **StorageClass**

Think of a PV as a **real disk** that exists in the cluster, backed by storage such as:

* AWS EBS
* GCP Persistent Disk
* NFS
* iSCSI
* Local disk

### Key Characteristics of PV

* Exists **independently of Pods**
* Has a **fixed capacity** (e.g., 5Gi, 10Gi)
* Has **access modes** (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
* Has a **reclaim policy** (Retain, Delete, Recycle)

### Example: PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

✅ This PV provides **5Gi storage** from the node filesystem.

---

## 🔹 What is a PersistentVolumeClaim (PVC)?

A **PersistentVolumeClaim (PVC)** is a **request for storage** made by a user or application.

Think of a PVC as:

> "I need 5Gi of storage with ReadWriteOnce access."

Kubernetes finds a matching PV and **binds** it to the PVC.

### Key Characteristics of PVC

* Namespace-scoped
* Requests storage size and access mode
* Abstracts storage details from developers
* Can trigger **dynamic provisioning**

### Example: PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

✅ This PVC requests **5Gi storage**.

---

## 🔁 PV–PVC Binding Process

1. PVC is created
2. Kubernetes searches for a **matching PV**
3. If found → **PV is bound to PVC**
4. PV becomes unavailable to other PVCs

Binding conditions:

* Same access mode
* PV capacity ≥ PVC request
* Same StorageClass (if defined)

---

## 🔹 Using PVC in a Pod

Pods never use PVs directly. They always use **PVCs**.

### Example: Pod Using PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-demo
```

📌 Pod → PVC → PV → Actual Storage

---

## 🧠 Simple Analogy

| Concept | Real-World Analogy         |
| ------- | -------------------------- |
| PV      | Actual house / flat        |
| PVC     | Rental agreement           |
| Pod     | Person living in the house |

---

## 🔍 PV vs PVC – Comparison Table

| Feature              | PV                          | PVC                 |
| -------------------- | --------------------------- | ------------------- |
| Scope                | Cluster-wide                | Namespace-specific  |
| Created by           | Admin / Dynamic Provisioner | Developer / User    |
| Purpose              | Provides storage            | Requests storage    |
| Lifecycle            | Independent of Pods         | Tied to application |
| Used directly by Pod | ❌ No                        | ✅ Yes               |

---

## ⚙️ Static vs Dynamic Provisioning

### Static Provisioning

* Admin manually creates PV
* PVC binds to existing PV

### Dynamic Provisioning

* PVC triggers automatic PV creation
* Requires **StorageClass**

---

## ✅ Key Takeaways

* **PV = Storage supply**
* **PVC = Storage demand**
* Pods always consume **PVCs, not PVs**
* Decouples storage from application lifecycle

---

If you want, next we can cover:

* StorageClass in depth
* Reclaim policies (Retain vs Delete)
* Real AWS EBS / NFS examples
