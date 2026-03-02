# What is a StatefulSet?

A **StatefulSet** is a Kubernetes workload API object used to manage **stateful applications**. Unlike Deployments, StatefulSets provide **stable network identity, stable storage, and ordered pod operations**, which are essential for applications that maintain state.

Typical use cases:

* Databases (MySQL, PostgreSQL)
* Distributed systems (Kafka, ZooKeeper, etcd)
* Applications that require stable hostnames or persistent disks

---

## Key Characteristics of StatefulSet

* **Stable Pod Identity**
  Each Pod gets a fixed name: `pod-name-0`, `pod-name-1`, etc.

* **Stable Network Identity**
  Pods get predictable DNS names using a **Headless Service**.

* **Stable Persistent Storage**
  Each Pod gets its own PersistentVolumeClaim (PVC).

* **Ordered Deployment & Scaling**
  Pods are created, deleted, and scaled **one by one in order**.

---

# What is a Deployment?

A **Deployment** is used to manage **stateless applications**. Pods are interchangeable and do not keep unique identities or storage.

Typical use cases:

* Web applications
* APIs
* Frontend services
* Microservices

---

# Deployment vs StatefulSet (Comparison)

| Feature          | Deployment          | StatefulSet           |
| ---------------- | ------------------- | --------------------- |
| Application Type | Stateless           | Stateful              |
| Pod Identity     | Random              | Fixed (index-based)   |
| Pod Name         | auto-generated      | predictable (`app-0`) |
| Storage          | Shared or ephemeral | Dedicated PVC per Pod |
| Scaling          | Parallel            | Ordered               |
| Network Identity | No guarantee        | Stable DNS            |
| Rolling Updates  | Fast & parallel     | Ordered & controlled  |

---

# Detailed Example: Deployment (Stateless App)

### Use Case

Running an Nginx web application where any Pod can serve traffic.

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Behavior

* Pods have random names like `nginx-deployment-7d9f8c9d5f-xkq2p`
* Pods are interchangeable
* No stable storage per Pod
* Scaling happens in parallel

---

# Detailed Example: StatefulSet (Stateful App)

### Use Case

Running a MySQL database where each Pod must keep its own data.

## Step 1: Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

## Step 2: StatefulSet YAML

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

### Behavior

* Pods are named `mysql-0`, `mysql-1`, `mysql-2`
* Each Pod gets its own PVC: `mysql-data-mysql-0`, etc.
* If a Pod restarts, it keeps the same data
* Pods start and stop in strict order

---

# Scaling Comparison

## Deployment Scaling

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

* Pods created/deleted in parallel

## StatefulSet Scaling

```bash
kubectl scale statefulset mysql --replicas=5
```

* Pods created one by one (`mysql-3` after `mysql-2` is ready)

---

# When to Use What?

### Use Deployment when:

* App is stateless
* Pods can be replaced anytime
* No need for stable identity

### Use StatefulSet when:

* App stores data
* Needs stable hostname or storage
* Ordered startup/shutdown is required

---

## One-Line Summary

* **Deployment = Stateless, scalable, interchangeable Pods**
* **StatefulSet = Stateful, ordered, stable identity & storage**
