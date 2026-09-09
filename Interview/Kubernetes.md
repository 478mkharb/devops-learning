# Kubernetes Interview Preparation — DevOps

A structured Kubernetes interview guide organized in **learning order**, from core architecture through workloads, scheduling, networking, storage, security, autoscaling, admission control, and troubleshooting.

This guide is designed for practical interview preparation. Each major topic includes:

- Interview questions
- Clear answers
- YAML/examples where useful
- Comparison tables
- Internal flow explanations
- Scenario-based questions
- Common interview traps
- Rapid-fire revision

> **Interview rule:** Do not memorize only definitions. Be able to explain **what the component does, why it exists, how it interacts with other components, and how you would troubleshoot it**.

---

# How to Use This README

Study in this order:

```text
Kubernetes Fundamentals
        ↓
Cluster Architecture
        ↓
Pods
        ↓
Scheduling
        ↓
Workloads
        ↓
Deployment Strategies
        ↓
Services
        ↓
DNS
        ↓
Networking / CNI
        ↓
Ingress
        ↓
Probes
        ↓
Resources / QoS
        ↓
HPA
        ↓
ConfigMap / Secrets
        ↓
Storage
        ↓
RBAC
        ↓
NetworkPolicy
        ↓
Troubleshooting
        ↓
Admission Control / Kyverno
        ↓
Advanced Comparisons
```

---

# 1. Kubernetes Fundamentals

## Q1. What is Kubernetes?

Kubernetes is an open-source container orchestration platform used to automate the deployment, scaling, networking, service discovery, and lifecycle management of containerized workloads.

It provides mechanisms for:

- Scheduling workloads onto nodes.
- Maintaining the desired number of application replicas.
- Service discovery.
- Load distribution.
- Rolling updates and rollbacks.
- Health checking.
- Configuration and secret management.
- Persistent storage integration.
- Access control.
- Automated reconciliation.

A simplified architecture is:

```text
Developer
   ↓
kubectl / API client
   ↓
Kubernetes API Server
   ↓
Cluster state + controllers + scheduler
   ↓
Worker nodes
   ↓
Pods
   ↓
Containers
```

---

## Q2. Why is Kubernetes called a declarative system?

In a declarative system, you specify the **desired state**, and Kubernetes continuously works to make the actual state match it.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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
          image: nginx:1.27
```

You say:

```text
I want 3 nginx Pods.
```

You do not normally specify:

```text
Create Pod 1
Create Pod 2
Create Pod 3
Restart Pod 2 if it dies
Create another Pod if Node A fails
```

Kubernetes controllers handle those actions.

---

## Q3. What is the desired state versus actual state?

| Desired State | Actual State |
|---|---|
| Defined by Kubernetes objects/configuration | What is currently running |
| Example: 3 replicas | Example: 2 Pods currently exist |
| User/controller declares it | Cluster observes it |
| Controllers try to enforce it | Changes continuously |

Example:

```text
Desired:
replicas = 3

Actual:
replicas = 2

Controller
   ↓
creates another Pod

Actual:
replicas = 3
```

This is the **reconciliation loop**.

---

## Q4. What is the reconciliation loop?

Kubernetes controllers continuously compare desired and observed state and take corrective action.

```text
Desired State
      ↓
Observe cluster
      ↓
Compare
      ↓
Difference?
   /       \
 No         Yes
 |           |
Wait       Act
             ↓
       Re-check state
```

Example:

```text
Deployment says: 3 replicas
Actual: 2 replicas
        ↓
Deployment/ReplicaSet controller
        ↓
Create Pod
        ↓
Actual becomes 3
```

This concept explains Kubernetes self-healing.

---

# 2. Kubernetes Cluster Architecture

## Q5. What are the major Kubernetes control-plane components?

The main components are:

| Component | Responsibility |
|---|---|
| API Server | Front door/API of the cluster |
| etcd | Persistent cluster state |
| Scheduler | Selects nodes for unscheduled Pods |
| Controller Manager | Runs control loops/controllers |

Worker nodes commonly contain:

| Component | Responsibility |
|---|---|
| kubelet | Manages Pods on the node |
| Container runtime | Runs containers |
| kube-proxy / networking dataplane | Service networking support |

---

## Q6. What does the Kubernetes API Server do?

The API Server is the central API endpoint through which Kubernetes clients and components communicate.

Typical request path:

```text
kubectl
  ↓
API Server
  ↓
Authentication
  ↓
Authorization
  ↓
Admission control
  ↓
Persist/read cluster state
```

The API Server is also responsible for exposing Kubernetes resources such as:

```text
Pod
Deployment
Service
ConfigMap
Secret
Node
Role
PVC
```

---

## Q7. What is etcd?

etcd is a distributed key-value store used by Kubernetes to persist cluster state.

It stores information such as:

- Kubernetes objects.
- Desired configuration.
- Cluster metadata.
- Resource definitions.
- Control-plane state.

Conceptually:

```text
API Server
    ↓
   etcd
    ↓
Cluster state
```

### Important interview point

etcd is **not the place where application containers run**.

It stores Kubernetes control-plane state.

---

## Q8. What does kube-scheduler do?

The scheduler selects an appropriate node for a Pod that does not yet have a node assignment.

Simplified flow:

```text
New Pod
  ↓
API Server
  ↓
Scheduler
  ↓
Find feasible nodes
  ↓
Score/select node
  ↓
Assign Pod to node
```

Scheduling considers constraints such as:

- CPU/memory requests.
- Node selectors.
- Node affinity.
- Taints/tolerations.
- Pod affinity/anti-affinity.
- Topology constraints.
- Other scheduling rules.

---

## Q9. What does kube-controller-manager do?

It runs Kubernetes controllers that continuously reconcile cluster state.

Examples include controllers responsible for:

- Deployments/ReplicaSets.
- Nodes.
- Jobs.
- Endpoints and other cluster resources.

Think:

```text
Desired state
     ↓
Controller
     ↓
Observe actual state
     ↓
Take corrective action
```

---

## Q10. What does kubelet do?

kubelet is the primary node agent.

It:

- Receives Pod specifications assigned to its node.
- Works with the container runtime.
- Ensures required containers are running.
- Reports node/Pod status to the API Server.
- Performs health-related lifecycle operations.

Simplified flow:

```text
API Server
   ↓
Pod assigned to Node
   ↓
kubelet
   ↓
CRI
   ↓
Container runtime
   ↓
Containers
```

---

## Q11. What is the Container Runtime Interface (CRI)?

CRI is the interface Kubernetes uses to communicate with container runtimes.

Examples of runtimes include:

- containerd
- CRI-O

Conceptually:

```text
kubelet
   ↓
CRI
   ↓
containerd / CRI-O
   ↓
containers
```

### Interview trap

Kubernetes does not require Docker Engine specifically as its runtime.

---

## Q12. What is kube-proxy?

Historically, kube-proxy implements Kubernetes Service networking behavior on nodes using mechanisms such as iptables or IPVS.

Modern Kubernetes networking implementations can also provide Service dataplane functionality in other ways, so avoid saying that every Kubernetes cluster must route every Service packet through kube-proxy.

Conceptually:

```text
Client
  ↓
Service virtual IP
  ↓
Service dataplane
  ↓
Backend Pod
```

---

## Q13. What is the difference between kubelet and kube-proxy?

| kubelet | kube-proxy |
|---|---|
| Node agent | Service networking component |
| Manages Pods/containers | Implements Service traffic behavior in traditional setups |
| Talks to container runtime through CRI | Programs networking rules/dataplane |
| Reports node/Pod status | Helps route Service traffic |

---

# 3. What Happens When You Create a Pod?

## Q14. Explain the complete Pod creation flow.

This is one of the most important interview questions.

Suppose you run:

```bash
kubectl apply -f pod.yaml
```

A simplified flow is:

```text
kubectl
   ↓
API Server
   ↓
Authentication / Authorization
   ↓
Admission
   ↓
Persist object in etcd
   ↓
Scheduler observes unscheduled Pod
   ↓
Scheduler selects node
   ↓
Pod assignment recorded through API Server
   ↓
kubelet watches assigned Pod
   ↓
CRI
   ↓
Container runtime
   ↓
Pod sandbox / networking
   ↓
Containers start
   ↓
kubelet reports status
   ↓
API Server
   ↓
etcd/status
```

### Interview follow-up

**Does kubectl directly contact kubelet?**

Normally no. `kubectl` communicates with the API Server. The control plane and kubelet then coordinate the workload.

---

# 4. Pods

## Q15. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain one or more tightly coupled containers that share:

- Network namespace.
- Pod IP.
- Ports/network identity.
- Volumes that are mounted into containers.

Most application Pods commonly contain one main application container.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
    - name: nginx
      image: nginx:1.27
```

---

## Q16. Why does Kubernetes use Pods instead of directly scheduling containers?

A Pod provides a shared execution boundary for one or more containers.

Containers in the same Pod can share:

- Network namespace.
- `localhost`.
- Volumes.
- Pod lifecycle.

Example:

```text
Pod
 ├── application container
 └── sidecar container
       |
       +--- shared network
       +--- shared volumes
```

This supports patterns such as:

- Sidecar logging.
- Proxy sidecars.
- Service mesh proxies.
- Supporting helper containers.

---

## Q17. Why is one container per Pod common?

Because a Pod is primarily a unit for **tightly coupled containers**, not a requirement to bundle unrelated applications.

If two containers have:

- Independent scaling needs.
- Independent lifecycle.
- Independent deployment cadence.

they often belong in separate Pods.

Example:

```text
Good:
Pod → application

Potential sidecar:
Pod → application + tightly coupled proxy

Usually poor:
Pod → unrelated application A + unrelated application B
```

---

## Q18. What is the difference between a Pod, container and Node?

| Object | Meaning |
|---|---|
| Container | Process/application runtime unit |
| Pod | Kubernetes deployment/scheduling unit containing one or more containers |
| Node | Machine/VM that runs Pods |

Hierarchy:

```text
Cluster
  ↓
Node
  ↓
Pod
  ↓
Container
```

A Node is **not** a container.

---

## Q19. What happens to a Pod when its Node fails?

It depends on the workload/controller.

A standalone Pod is not automatically recreated by a Deployment controller because it is not managed by a Deployment.

For a Deployment:

```text
Node fails
   ↓
Pods on node become unavailable
   ↓
ReplicaSet/Deployment controller sees fewer available replicas
   ↓
Scheduler schedules replacement Pods on healthy nodes
```

This is why production applications are normally managed by controllers rather than manually created standalone Pods.

---

## Q20. Why are Pod IPs considered ephemeral?

Pod IPs are tied to Pod lifecycle.

If a Pod is deleted and a replacement is created:

```text
old Pod → 10.x.x.10
delete
new Pod → 10.x.x.27
```

The replacement does not necessarily receive the same IP.

Therefore applications should normally communicate through a **Service** rather than hardcoding Pod IPs.

---

# 5. Scheduling

## Q21. What is nodeSelector?

`nodeSelector` is a simple way to constrain a Pod to nodes having specific labels.

Label:

```bash
kubectl label node node1 disk=ssd
```

Pod:

```yaml
spec:
  nodeSelector:
    disk: ssd
```

Only nodes with:

```text
disk=ssd
```

are eligible.

---

## Q22. What is node affinity?

Node affinity provides more expressive Pod-to-node placement rules.

Example:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disk
              operator: In
              values:
                - ssd
```

Types include:

```text
requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution
```

### Required vs preferred

| Type | Meaning |
|---|---|
| Required | Must satisfy rule |
| Preferred | Scheduler tries to satisfy rule |

---

## Q23. What is Pod affinity?

Pod affinity places a Pod near other Pods based on labels/topology.

Example use case:

```text
Application Pod
     ↓
Prefer same zone as
     ↓
Cache Pod
```

This can reduce latency for tightly coupled workloads.

---

## Q24. What is Pod anti-affinity?

Pod anti-affinity tries to prevent Pods from being placed near selected Pods.

Typical use:

```text
web-0 → node A
web-1 → node B
web-2 → node C
```

This improves resilience by reducing the chance that one node failure removes all replicas.

---

## Q25. What are taints and tolerations?

A **taint** is placed on a node to repel Pods.

A **toleration** is placed on a Pod to allow it to be scheduled onto a tainted node.

Example taint:

```bash
kubectl taint nodes node1 dedicated=database:NoSchedule
```

Pod:

```yaml
tolerations:
  - key: dedicated
    operator: Equal
    value: database
    effect: NoSchedule
```

Mental model:

```text
Taint
  ↓
"Do not schedule here"

Toleration
  ↓
"This Pod is allowed here"
```

A toleration does **not** force scheduling onto the node.

---

## Q26. What are `NoSchedule`, `PreferNoSchedule`, and `NoExecute`?

| Effect | Meaning |
|---|---|
| `NoSchedule` | New Pods without matching toleration are not scheduled |
| `PreferNoSchedule` | Scheduler tries to avoid the node |
| `NoExecute` | Affects scheduling and can evict existing non-tolerating Pods |

### Important distinction

`NoSchedule` primarily affects **new scheduling**.

`NoExecute` can affect **existing Pods** on the node.

---

## Q27. How would you ensure replicas run on different nodes?

Use Pod anti-affinity or topology-aware scheduling.

Example:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: web
        topologyKey: kubernetes.io/hostname
```

This says replicas should not be placed on the same hostname domain.

---

# 6. Workloads

## Q28. What is a ReplicaSet?

A ReplicaSet ensures that a specified number of matching Pods exist.

```yaml
spec:
  replicas: 3
```

Conceptually:

```text
ReplicaSet
   ↓
Desired = 3
   ↓
Pod Pod Pod
```

If one Pod disappears:

```text
3 → 2
   ↓
ReplicaSet creates replacement
   ↓
3
```

---

## Q29. What is a Deployment?

A Deployment manages stateless application rollout and typically manages ReplicaSets.

Hierarchy:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers
```

Deployment features include:

- Scaling.
- Rolling updates.
- Rollbacks.
- Revision history.
- Replica management.

---

## Q30. What happens when you change a Deployment image?

Example:

```yaml
containers:
  - name: app
    image: myapp:v2
```

Changing from `v1` to `v2` normally creates a new Deployment revision and ReplicaSet.

Simplified:

```text
Deployment
    ↓
New ReplicaSet
    ↓
New Pods v2
    ↓
Old ReplicaSet scaled down
```

With a rolling update, old and new Pods can coexist temporarily.

---

## Q31. What is a StatefulSet?

StatefulSet manages applications requiring stable identity and/or stable storage characteristics.

Typical use cases:

- Databases.
- Distributed databases.
- Stateful clustered applications.

Characteristics can include:

- Stable Pod names.
- Stable network identity.
- Ordered operations.
- Persistent storage association.

Example:

```text
db-0
db-1
db-2
```

---

## Q32. What is a DaemonSet?

DaemonSet ensures that a Pod runs on every eligible node, or on every eligible node matching scheduling constraints.

Typical uses:

- Node monitoring agents.
- Log collectors.
- Security agents.
- CNI-related components.

When a new eligible node joins:

```text
New Node
   ↓
DaemonSet controller
   ↓
Daemon Pod scheduled
```

---

## Q33. What is a Job?

A Job manages a workload intended to run to completion.

Example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migration
          image: myapp:migrate
```

A Job is different from a Deployment because success means **completion**, not continuous service availability.

---

## Q34. What is a CronJob?

CronJob creates Jobs on a schedule.

Example:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: backup
              image: backup:latest
```

Concept:

```text
CronJob
   ↓
Scheduled time
   ↓
Job
   ↓
Pod
```

---

## Q35. Compare Deployment, StatefulSet, DaemonSet, Job and CronJob.

| Workload | Main purpose |
|---|---|
| Deployment | Stateless long-running applications |
| StatefulSet | Stateful applications with stable identity/storage |
| DaemonSet | One Pod per eligible node |
| Job | Run-to-completion workload |
| CronJob | Scheduled Jobs |
| ReplicaSet | Maintain desired number of matching Pods |

---

# 7. Deployment Strategies

## Q36. What is a RollingUpdate?

RollingUpdate gradually replaces old Pods with new Pods.

```text
v1 v1 v1
 ↓
v2 v1 v1
 ↓
v2 v2 v1
 ↓
v2 v2 v2
```

Important settings:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

---

## Q37. What are `maxSurge` and `maxUnavailable`?

| Setting | Meaning |
|---|---|
| `maxSurge` | Maximum additional Pods above desired replicas during rollout |
| `maxUnavailable` | Maximum Pods allowed to be unavailable during rollout |

Example:

```yaml
replicas: 4

maxSurge: 1
maxUnavailable: 0
```

The controller can temporarily run up to 5 Pods while keeping all 4 desired replicas available, subject to readiness and other constraints.

---

## Q38. How do you rollback a Deployment?

Check history:

```bash
kubectl rollout history deployment/myapp
```

Rollback:

```bash
kubectl rollout undo deployment/myapp
```

Check status:

```bash
kubectl rollout status deployment/myapp
```

Conceptually:

```text
Revision 1 → v1
Revision 2 → v2
Revision 3 → v3

rollback
   ↓
Revision 2 / selected revision
```

---

# 8. Services

## Q39. Why do we need a Kubernetes Service?

Pods are ephemeral and their IP addresses can change.

A Service provides a stable logical endpoint for a set of Pods.

```text
Client
   ↓
Service
   ↓
Pod Pod Pod
```

The Service uses selectors to identify backend Pods.

---

## Q40. What is ClusterIP?

ClusterIP is the default Service type.

It provides internal cluster access.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

Traffic:

```text
Pod A
  ↓
backend:80
  ↓
backend Pod:8080
```

---

## Q41. What is NodePort?

NodePort exposes a Service through a port on each eligible node.

Conceptually:

```text
Client
  ↓
NodeIP:NodePort
  ↓
Service
  ↓
Pod
```

It is commonly useful as a building block for external exposure, although production architectures often use a cloud LoadBalancer or Ingress.

---

## Q42. What is a LoadBalancer Service?

A `LoadBalancer` Service requests an external load-balancing integration from the environment/cloud provider.

Conceptually:

```text
Internet
   ↓
Cloud Load Balancer
   ↓
Service
   ↓
Pods
```

The exact implementation depends on the Kubernetes environment and cloud integration.

---

## Q43. What are `port` and `targetPort`?

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Meaning:

```text
Client
  ↓
Service port 80
  ↓
Pod/container port 8080
```

| Field | Meaning |
|---|---|
| `port` | Service port |
| `targetPort` | Backend Pod port |

---

## Q44. What is a Service selector?

A selector determines which Pods are backends for a Service.

Service:

```yaml
selector:
  app: backend
```

Pod:

```yaml
labels:
  app: backend
```

They match.

If the Pod has:

```yaml
labels:
  app: api
```

the Service does not select it.

---

## Q45. What are Endpoints and EndpointSlices?

They represent backend endpoints associated with Services.

Modern Kubernetes uses **EndpointSlice** as the scalable mechanism for representing service endpoints.

Conceptually:

```text
Service selector
      ↓
Matching Pods
      ↓
EndpointSlice
      ↓
Service dataplane
```

---

## Q46. What happens if a Service has no endpoints?

A likely cause is a selector mismatch.

Debug:

```bash
kubectl get svc
kubectl describe svc backend
kubectl get endpoints backend
kubectl get endpointslices
kubectl get pods --show-labels
```

Check:

```text
Service selector
       ↓
Pod labels
       ↓
Do they match?
```

---

# 9. Kubernetes DNS / CoreDNS

## Q47. What is CoreDNS?

CoreDNS is commonly used as the DNS service inside a Kubernetes cluster.

It allows workloads to resolve Kubernetes Services and other DNS records.

Conceptually:

```text
Pod
 ↓
DNS query
 ↓
CoreDNS
 ↓
Service DNS record
 ↓
Service IP
```

---

## Q48. How do Pods communicate with a Service using DNS?

Example Service:

```text
backend
```

Within the same namespace, a Pod can typically use:

```text
backend
```

Cross-namespace:

```text
backend.production
```

Fully qualified form:

```text
backend.production.svc.cluster.local
```

Conceptually:

```text
Application
   ↓
backend.production.svc.cluster.local
   ↓
CoreDNS
   ↓
Service
   ↓
Backend Pods
```

---

## Q49. How do you troubleshoot Kubernetes DNS?

Useful commands:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

From an application/debug Pod:

```bash
nslookup backend
nslookup backend.production.svc.cluster.local
```

Check:

- CoreDNS Pods.
- CoreDNS Service.
- Pod `/etc/resolv.conf`.
- NetworkPolicy.
- CNI/network connectivity.
- Service existence.

---

# 10. Kubernetes Networking

## Q50. What is the Kubernetes Pod networking model?

A Kubernetes networking implementation generally provides each Pod with an IP address that is routable according to the cluster's network design.

Containers within the same Pod share the Pod network namespace.

```text
Pod
 ├── container A
 └── container B
       |
       +--- same network namespace
       +--- same Pod IP
       +--- localhost
```

---

## Q51. What is the pause container / Pod sandbox?

The Pod sandbox establishes the Pod's execution/networking environment.

A simplified model is:

```text
Pod Sandbox
   ↓
Network namespace
   ↓
Pod IP
   ↓
Application containers join Pod network
```

The exact implementation is runtime/CNI dependent, but the important concept is that containers in a Pod share the Pod's network namespace.

---

## Q52. What is CNI?

CNI stands for **Container Network Interface**.

It defines a mechanism through which container runtimes and networking plugins configure container networking.

CNI implementations/plugins can handle things such as:

- Pod IP allocation.
- Network interfaces.
- Routes.
- Connectivity.
- Network policy support, depending on implementation.

Common Kubernetes networking solutions include:

| Solution | General characteristic |
|---|---|
| Flannel | Simple cluster networking |
| Calico | Networking + NetworkPolicy |
| Cilium | eBPF-based networking/security |
| AWS VPC CNI | AWS-native Pod networking |

---

## Q53. What is a veth pair?

A veth pair is a pair of connected virtual Ethernet interfaces.

A simplified Pod networking setup can look like:

```text
Pod network namespace
       |
    veth-pod
       |
    veth-host
       |
    Linux bridge / host networking
```

This allows traffic to move between the Pod network namespace and the node's networking environment.

---

## Q54. How does same-node Pod-to-Pod communication work?

A simplified model:

```text
Pod A
  ↓
Pod network interface
  ↓
Node networking
  ↓
Pod B network interface
  ↓
Pod B
```

The exact path depends on the CNI implementation.

The important interview point is that Kubernetes networking is implemented by the cluster's network plugin rather than by the application containers themselves.

---

## Q55. How does cross-node Pod communication work?

A simplified model:

```text
Pod A on Node 1
       ↓
Node 1 networking
       ↓
CNI / routing / overlay or native networking
       ↓
Node 2
       ↓
Pod B
```

Implementations may use:

- Native routing.
- Overlay networks.
- Encapsulation such as VXLAN.
- eBPF dataplanes.
- Cloud-native VPC networking.

Do not assume every CNI uses the same packet path.

---

# 11. Ingress

## Q56. What is Kubernetes Ingress?

Ingress is an API resource for defining HTTP/HTTPS routing rules into cluster services.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

Ingress itself is a **configuration object**.

---

## Q57. What is an Ingress Controller?

The Ingress Controller is the implementation that watches Ingress resources and configures/operates the traffic-handling dataplane.

Examples of controller technologies include:

- NGINX-based controllers.
- Traefik.
- HAProxy.
- Cloud-provider-specific controllers.

Mental model:

```text
Ingress
  =
Routing rules

Ingress Controller
  =
Implementation that enforces those rules
```

---

## Q58. Can Ingress work without an Ingress Controller?

An Ingress resource by itself does not provide traffic handling.

You need an appropriate controller or equivalent implementation.

```text
Ingress resource
      ↓
Controller watches it
      ↓
Controller configures proxy/load balancer
      ↓
Traffic reaches Service
```

---

## Q59. What is the difference between Ingress and LoadBalancer Service?

| Ingress | LoadBalancer Service |
|---|---|
| HTTP/HTTPS routing rules | Exposes a Service externally |
| Can route by host/path | Primarily exposes one Service |
| Can consolidate multiple routes | Usually one load-balancer service endpoint |
| Needs controller/implementation | Needs environment/cloud integration |

Example:

```text
app.example.com → frontend
api.example.com → backend
```

One ingress layer can route to multiple Services.

---

## Q60. What is host-based routing?

Traffic is routed according to hostname.

```text
app.example.com → frontend Service
api.example.com → backend Service
```

Example:

```yaml
rules:
  - host: app.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: frontend
              port:
                number: 80

  - host: api.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: backend
              port:
                number: 8080
```

---

## Q61. What is path-based/fanout routing?

Traffic is routed according to URL path.

```text
example.com/
      ↓
frontend

example.com/api
      ↓
backend
```

This is useful when multiple applications share a hostname.

---

# 12. Probes

## Q62. What is a liveness probe?

A liveness probe checks whether a container should be considered alive.

If the liveness check repeatedly fails, kubelet may restart the container.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
```

---

## Q63. What is a readiness probe?

Readiness determines whether a Pod should receive normal Service traffic.

If readiness fails:

```text
Pod may remain Running
       ↓
Pod becomes NotReady
       ↓
Removed from Service endpoints
       ↓
Traffic stops going to it
```

It normally does **not** restart the container merely because readiness failed.

---

## Q64. What is a startup probe?

Startup probe is useful for applications that take a long time to initialize.

While the startup probe is failing, Kubernetes can delay the normal liveness/readiness probing behavior according to the configured probe lifecycle.

Example:

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

---

## Q65. Compare startup, liveness and readiness probes.

| Probe | Main question | Typical consequence |
|---|---|---|
| Startup | Has application finished starting? | Protects slow-starting apps from premature liveness failure |
| Liveness | Is the container still alive? | Failed checks can trigger restart |
| Readiness | Can the Pod receive traffic? | Failed checks remove it from normal Service endpoints |

### Classic interview question

**Pod is Running but users cannot access the application. What do you check?**

Check:

```text
Pod status
   ↓
Readiness
   ↓
Service selector
   ↓
EndpointSlice
   ↓
Service port/targetPort
   ↓
NetworkPolicy
   ↓
Ingress
```

---

# 13. Resource Requests, Limits and QoS

## Q66. What is a CPU/memory request?

A resource request tells the scheduler how much resource the Pod requires for scheduling purposes.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

The scheduler uses requests when determining whether a node has enough allocatable capacity.

---

## Q67. What is a resource limit?

A resource limit establishes an upper boundary/enforcement value for the container's resource usage, subject to Kubernetes/runtime behavior.

Example:

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
```

### Request vs limit

| Request | Limit |
|---|---|
| Used for scheduling | Upper resource boundary/enforcement |
| Helps determine node placement | Controls resource consumption behavior |
| Can be lower than limit | Usually equal to or greater than request |

---

## Q68. What is OOMKilled?

OOMKilled means a container was killed because it exceeded an applicable memory limit or the node/runtime experienced memory pressure leading to an out-of-memory kill.

Check:

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o wide
kubectl logs <pod> --previous
```

Typical symptom:

```text
Last State:
  Terminated
  Reason: OOMKilled
```

---

## Q69. What are Kubernetes QoS classes?

Kubernetes assigns Pods a QoS class based on their resource requests/limits.

Main classes:

| QoS | General condition |
|---|---|
| Guaranteed | CPU and memory requests/limits meet Guaranteed criteria |
| Burstable | Has requests/limits but does not meet Guaranteed criteria |
| BestEffort | No CPU/memory requests or limits |

QoS affects behavior during resource pressure, but eviction is not simply "BestEffort always first." Actual usage, requests, node pressure and eviction rules matter.

---

# 14. Horizontal Pod Autoscaler

## Q70. What is HPA?

Horizontal Pod Autoscaler automatically adjusts the number of Pod replicas based on observed metrics.

Conceptually:

```text
Metrics
   ↓
HPA
   ↓
Desired replicas
   ↓
Deployment
   ↓
More/fewer Pods
```

Example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

---

## Q71. What happens when CPU utilization increases above the HPA target?

Example:

```text
Target CPU = 60%
Actual CPU = 90%
        ↓
HPA calculates desired replicas
        ↓
Deployment replica count increases
        ↓
More Pods scheduled
```

The exact scaling decision also depends on HPA configuration and metric sampling/behavior.

---

## Q72. What does HPA require?

For common resource-based HPA, the cluster needs a working metrics source such as Metrics Server and appropriate resource requests on containers for utilization-based CPU/memory calculations.

Troubleshoot:

```bash
kubectl get apiservice
kubectl top pods
kubectl top nodes
kubectl describe hpa <name>
```

---

# 15. ConfigMap and Secrets

## Q73. What is a ConfigMap?

ConfigMap stores non-sensitive configuration data.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  APP_MODE: "production"
```

Consume as environment variables:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

---

## Q74. What is a Secret?

Secret stores data intended to be sensitive, such as:

- Passwords.
- Tokens.
- Credentials.
- Certificates.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: app
  password: example-password
```

### Important security point

A Kubernetes Secret is not automatically equivalent to a secure external secrets-management system. Protect RBAC access and configure encryption at rest where required.

---

## Q75. ConfigMap vs Secret?

| ConfigMap | Secret |
|---|---|
| Non-sensitive configuration | Sensitive data |
| Log level | Password/token |
| Feature flags | Credentials |
| Application settings | Certificates/keys |

Both can be consumed as:

- Environment variables.
- Mounted volumes.

---

# 16. Storage

## Q76. What is a PersistentVolume (PV)?

A PersistentVolume is a cluster storage resource that represents storage made available to Kubernetes.

It can be backed by different storage systems depending on the environment.

Example:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
```

---

## Q77. What is a PersistentVolumeClaim (PVC)?

A PVC is a request for storage made by a workload/user.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Relationship:

```text
PVC
 ↓
PV
 ↓
Storage backend
```

---

## Q78. What is a StorageClass?

StorageClass describes a class of storage and typically enables dynamic provisioning through a storage provisioner.

Conceptually:

```text
PVC
 ↓
StorageClass
 ↓
Provisioner
 ↓
Storage volume
 ↓
PV
```

This avoids manually creating every PV in many environments.

---

## Q79. Static vs dynamic provisioning?

| Static provisioning | Dynamic provisioning |
|---|---|
| Admin creates PV beforehand | PVC triggers provisioning |
| Manual | Automated |
| PV must already exist | StorageClass/provisioner creates storage |
| Useful for pre-existing storage | Common for cloud-native environments |

---

## Q80. What is `emptyDir`?

`emptyDir` is temporary storage associated with a Pod.

```yaml
volumes:
  - name: cache
    emptyDir: {}
```

Its contents exist while the Pod exists on the node.

If the Pod is deleted and recreated, the `emptyDir` data is not preserved.

Typical use:

- Temporary cache.
- Scratch space.
- Sharing files between containers in the same Pod.

---

## Q81. What are common access modes?

Common access modes include:

| Access mode | Meaning |
|---|---|
| RWO | ReadWriteOnce |
| ROX | ReadOnlyMany |
| RWX | ReadWriteMany |
| RWOP | ReadWriteOncePod |

The exact support depends on the storage backend/driver.

---

## Q82. How does StatefulSet create persistent storage?

StatefulSets can use `volumeClaimTemplates`.

Conceptually:

```text
StatefulSet
   ↓
volumeClaimTemplates
   ↓
PVC for db-0
PVC for db-1
PVC for db-2
```

This provides stable storage association for individual StatefulSet Pods.

---

## Q83. A PVC is stuck in Pending. How do you troubleshoot it?

Check:

```bash
kubectl get pvc
kubectl describe pvc <pvc>
kubectl get storageclass
kubectl get pv
```

Investigate:

```text
PVC
 ↓
StorageClass exists?
 ↓
Provisioner working?
 ↓
Requested size supported?
 ↓
Access mode supported?
 ↓
Topology constraints?
 ↓
Backend/cloud volume errors?
```

---

# 17. RBAC

## Q84. What is Kubernetes RBAC?

RBAC stands for **Role-Based Access Control**.

It controls what subjects can do with Kubernetes resources.

Core relationship:

```text
User / Group / ServiceAccount
             ↓
      RoleBinding
             ↓
           Role
             ↓
        Permissions
```

---

## Q85. What is a Role?

A Role defines permissions within a namespace.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: app
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

---

## Q86. What is a ClusterRole?

ClusterRole defines permissions that can be used at cluster scope or referenced by bindings in a namespace.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

---

## Q87. Role vs ClusterRole?

| Role | ClusterRole |
|---|---|
| Namespace-scoped permission definition | Cluster-scoped permission definition |
| Used for namespace resources | Can describe cluster-scoped permissions |
| Bound with RoleBinding | Can be bound with RoleBinding or ClusterRoleBinding |

Important nuance:

A `RoleBinding` can reference a `ClusterRole`. The resulting permission is still limited by the namespace scope of the RoleBinding.

---

## Q88. What is RoleBinding?

RoleBinding grants the permissions of a Role or ClusterRole to subjects within a namespace.

```text
Subject
   ↓
RoleBinding
   ↓
Role / ClusterRole
```

Subjects can include:

- User.
- Group.
- ServiceAccount.

---

## Q89. What is ClusterRoleBinding?

ClusterRoleBinding grants a ClusterRole's permissions at cluster scope.

```text
Subject
   ↓
ClusterRoleBinding
   ↓
ClusterRole
   ↓
Cluster-wide authorization
```

Use it carefully because excessive cluster-wide permissions are a security risk.

---

## Q90. What is a ServiceAccount?

A ServiceAccount is an identity used by workloads running in Kubernetes.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: app
```

Pod:

```yaml
spec:
  serviceAccountName: app-sa
```

The workload can then authenticate to Kubernetes APIs according to the permissions granted to that ServiceAccount.

---

## Q91. How do you check whether an identity has permission?

Use:

```bash
kubectl auth can-i get pods
```

For a ServiceAccount:

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:app:app-sa
```

This is an extremely useful troubleshooting command.

---

# 18. NetworkPolicy

## Q92. What is NetworkPolicy?

NetworkPolicy defines rules controlling allowed network traffic to/from selected Pods.

A policy can control:

- Ingress.
- Egress.
- Source/destination Pods.
- Namespaces.
- IP blocks.

Example conceptual architecture:

```text
Frontend
   ↓ allowed
Backend
   ↓ allowed
Database

Other workloads
   ↓ denied
Database
```

---

## Q93. What is a default-deny NetworkPolicy?

A common security pattern is to start with deny-all and then explicitly allow required traffic.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: app
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Then add specific allow policies.

---

## Q94. Does creating a NetworkPolicy automatically enforce network restrictions?

No.

The Kubernetes network implementation/CNI must support NetworkPolicy enforcement.

This is an important interview distinction:

```text
NetworkPolicy object
       ≠
Automatic enforcement by every possible network implementation
```

---

# 19. Troubleshooting

## Q95. How do you troubleshoot a Pod in `Pending`?

Start with:

```bash
kubectl get pod <pod>
kubectl describe pod <pod>
kubectl get nodes
kubectl describe nodes
```

Look at Events.

Common causes:

| Cause | Example |
|---|---|
| Insufficient resources | CPU/memory unavailable |
| Taint | Pod lacks toleration |
| NodeSelector | No matching node |
| Affinity | Scheduling constraint impossible |
| PVC | Storage not bound |
| Node issue | No Ready nodes |

Mental flow:

```text
Pending
 ↓
kubectl describe pod
 ↓
Events
 ↓
Scheduling?
 ↓
Resources / taints / affinity / PVC
```

---

## Q96. What is CrashLoopBackOff and how do you troubleshoot it?

CrashLoopBackOff means the container is repeatedly failing and Kubernetes is applying increasing restart delays.

Check:

```bash
kubectl get pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

Common causes:

- Application crash.
- Wrong command.
- Bad environment variable.
- Missing configuration.
- Dependency unavailable.
- Permission error.
- Probe failure.

---

## Q97. What is ImagePullBackOff?

It means Kubernetes is having trouble pulling the container image and is backing off between retries.

Check:

```bash
kubectl describe pod <pod>
```

Look for events such as:

```text
Failed to pull image
ErrImagePull
ImagePullBackOff
```

Common causes:

- Wrong image name.
- Wrong tag.
- Private registry authentication.
- Registry/network issue.
- Image doesn't exist.

---

## Q98. A Pod is Running but application is unavailable. What do you check?

Do not assume `Running` means healthy.

Use:

```bash
kubectl get pod
kubectl describe pod
kubectl logs
kubectl get svc
kubectl get endpointslices
```

Check:

```text
Running?
   ↓
Ready?
   ↓
Readiness probe?
   ↓
Service selector?
   ↓
EndpointSlice?
   ↓
port/targetPort?
   ↓
NetworkPolicy?
   ↓
Ingress?
   ↓
Application itself?
```

---

## Q99. A Service exists but traffic doesn't reach Pods. What do you check?

First:

```bash
kubectl describe svc <service>
kubectl get endpointslices
kubectl get pods --show-labels
```

Verify:

```text
Service selector
       =
Pod labels
```

Then check:

- Service port.
- `targetPort`.
- Pod readiness.
- EndpointSlice.
- NetworkPolicy.
- Application listening port.
- Ingress/load balancer if traffic is external.

---

## Q100. How do you troubleshoot NodeNotReady?

Start with:

```bash
kubectl get nodes
kubectl describe node <node>
```

Then inspect the node itself:

```text
kubelet
container runtime
CPU/memory/disk pressure
network connectivity
certificates
system services
CNI
```

Typical checks on the node:

```bash
systemctl status kubelet
systemctl status containerd
journalctl -u kubelet
journalctl -u containerd
```

Also inspect:

```bash
kubectl describe node <node>
```

for conditions such as:

- MemoryPressure.
- DiskPressure.
- PIDPressure.
- NetworkUnavailable.
- Ready condition.

---

# 20. Admission Control

## Q101. What is admission control?

Admission control is the stage in the API Server request flow where Kubernetes can validate or modify API requests before they are persisted.

Simplified:

```text
kubectl
   ↓
API Server
   ↓
Authentication
   ↓
Authorization
   ↓
Admission Control
   ↓
Persist
   ↓
etcd
```

---

## Q102. What is mutating admission?

A mutating admission controller can modify an incoming API request.

Example:

```text
Developer creates Pod
        ↓
Mutating webhook
        ↓
Adds sidecar
        ↓
Adds label
        ↓
Request continues
```

---

## Q103. What is validating admission?

A validating admission controller checks whether a request complies with policy.

Example:

```text
Pod requests privileged mode
        ↓
Validation policy
        ↓
Reject
```

It normally does not modify the object.

---

## Q104. Mutating vs validating admission?

| Mutating | Validating |
|---|---|
| Can modify request | Validates request |
| Inject sidecar | Reject forbidden configuration |
| Add labels/defaults | Enforce security rules |
| Can transform object | Typically accepts/rejects |

---

# 21. Kyverno

## Q105. What is Kyverno?

Kyverno is a Kubernetes-native policy engine used to validate, mutate, generate, and otherwise govern Kubernetes resources.

Conceptually:

```text
kubectl apply
     ↓
API Server
     ↓
Kyverno policy
     ↓
Validate / Mutate / Generate
     ↓
Allow or Reject
```

---

## Q106. What can Kyverno policies do?

Common policy types include:

| Policy behavior | Example |
|---|---|
| Validate | Require labels |
| Mutate | Add default labels |
| Generate | Create related resources |
| Verify images | Enforce trusted images/signatures |

Examples of policies:

- Require `owner` label.
- Prevent privileged containers.
- Disallow `latest` tags.
- Require resource limits.
- Restrict registries.
- Require non-root containers.
- Generate NetworkPolicies.

---

## Q107. Why use Kyverno instead of manually checking YAML?

Manual review is inconsistent and does not scale.

With policy:

```text
Developer
   ↓
kubectl apply
   ↓
Policy automatically evaluated
   ↓
Violation
   ↓
Request rejected
```

This moves governance closer to the point where resources enter the cluster.

---

# 22. Scenario-Based Interview Questions

## Q108. A Deployment has 3 replicas, but only 2 Pods are running. What do you investigate?

Check:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
kubectl describe deployment <name>
kubectl describe rs <name>
```

Then:

```text
Deployment desired = 3
        ↓
ReplicaSet desired = 3?
        ↓
Pod exists?
        ↓
Pod Pending?
        ↓
Pod crashing?
        ↓
Image problem?
        ↓
Scheduling problem?
```

---

## Q109. A Pod is Pending because of a taint. How do you fix it?

First determine whether the Pod should actually run on that node.

If yes, add the appropriate toleration.

If no, don't blindly add a toleration. Find a suitable node or fix the scheduling requirement.

---

## Q110. A Service has no endpoints although Pods are Running. What is the likely issue?

A common cause is a selector mismatch.

Example:

```text
Service selector:
app=backend

Pod:
app=api
```

No match:

```text
Service
  ↓
No matching Pods
  ↓
No EndpointSlice entries
```

Check:

```bash
kubectl describe svc backend
kubectl get pods --show-labels
kubectl get endpointslices
```

---

## Q111. Readiness probe is failing. Will Kubernetes restart the Pod?

Normally no.

Readiness controls whether the Pod receives Service traffic.

```text
Readiness fails
      ↓
Pod becomes NotReady
      ↓
Removed from Service endpoints
```

Liveness failure is the probe associated with restarting an unhealthy container.

---

## Q112. Liveness probe is failing but the application is actually just slow to start. What is the likely design problem?

The application may need a **startup probe** or better startup/liveness configuration.

Without one, liveness can begin failing before the application has finished initialization.

---

## Q113. HPA is not scaling your Deployment. What do you check?

Check:

```bash
kubectl get hpa
kubectl describe hpa <hpa>
kubectl top pods
kubectl top nodes
```

Then verify:

- Metrics source is working.
- Resource requests exist if using utilization-based resource metrics.
- HPA target references the correct Deployment.
- Current/desired replicas.
- Metric values are available.
- Scaling limits aren't preventing the expected result.

---

## Q114. A PVC remains Pending. What do you investigate?

```text
PVC
 ↓
StorageClass?
 ↓
Provisioner?
 ↓
PV?
 ↓
Capacity?
 ↓
Access mode?
 ↓
Topology?
 ↓
Storage backend?
```

Commands:

```bash
kubectl describe pvc <pvc>
kubectl get storageclass
kubectl get pv
```

---

## Q115. An application can access the frontend but not the database. What do you check?

Think layer by layer:

```text
Application
   ↓
Service
   ↓
DNS
   ↓
EndpointSlice
   ↓
NetworkPolicy
   ↓
Database Pod/Service
   ↓
Database listener
```

Check:

```bash
kubectl get svc
kubectl get endpointslices
kubectl get networkpolicy
kubectl exec -it <pod> -- nslookup db
kubectl exec -it <pod> -- <connectivity-test>
```

---

# 23. Advanced Comparison Questions

## Q116. Deployment vs StatefulSet?

| Deployment | StatefulSet |
|---|---|
| Stateless applications | Stateful applications |
| Pods are interchangeable | Pods have stable identity |
| No stable Pod identity required | Stable names/network identity |
| Common web/API workloads | Databases/distributed systems |
| Scaling generally interchangeable | Ordered/stable semantics |

---

## Q117. Deployment vs DaemonSet?

| Deployment | DaemonSet |
|---|---|
| Desired number of replicas | Pod on each eligible node |
| Application workloads | Node-level agents |
| 3 replicas can be on selected nodes | Typically one per eligible node |
| Scaling based on replica count | Scales with eligible nodes |

---

## Q118. ReplicaSet vs Deployment?

| ReplicaSet | Deployment |
|---|---|
| Maintains replicas | Manages rollout/lifecycle |
| Lower-level controller | Higher-level workload abstraction |
| Can exist independently | Normally manages ReplicaSets |
| Limited rollout functionality | Rolling updates/rollback/revisions |

---

## Q119. Service vs Ingress?

| Service | Ingress |
|---|---|
| Stable endpoint for Pods | HTTP/HTTPS routing layer |
| Service discovery/load distribution | Host/path routing |
| ClusterIP/NodePort/LoadBalancer | Requires controller implementation |
| Works directly with selected Pods | Routes to Services |

---

## Q120. Service vs EndpointSlice?

| Service | EndpointSlice |
|---|---|
| Stable logical abstraction | Backend endpoint representation |
| Defines how clients access application | Tracks selected backend endpoints |
| Has selector/ports/type | Contains endpoint addresses and metadata |

---

## Q121. ConfigMap vs Secret?

```text
ConfigMap → non-sensitive configuration
Secret    → sensitive data
```

But both require appropriate RBAC and operational security.

---

## Q122. NodeSelector vs Node Affinity?

| nodeSelector | Node affinity |
|---|---|
| Simple | More expressive |
| Exact label matching | Operators/required/preferred rules |
| Easy to understand | More complex scheduling requirements |

---

## Q123. Taint/Toleration vs Affinity?

| Taint/Toleration | Affinity |
|---|---|
| Controls whether Pods are repelled/allowed | Expresses placement preference/requirements |
| Primarily node-side repelling mechanism | Pod-side scheduling constraints |
| Taint repels | Affinity attracts/selects |
| Toleration permits but does not force | Affinity can require/prefer placement |

---

# 24. Docker Swarm vs Kubernetes

## Q124. Kubernetes vs Docker Swarm?

| Kubernetes | Docker Swarm |
|---|---|
| Larger ecosystem | Simpler operational model |
| Rich workload abstractions | Simpler service model |
| Advanced scheduling | Simpler scheduling |
| Strong extensibility | More limited ecosystem |
| Extensive networking/storage/security ecosystem | Easier initial setup |
| Widely adopted for large-scale orchestration | Useful for simpler container orchestration |

### Interview answer

> Kubernetes is generally preferred for complex production orchestration because of its ecosystem, extensibility, workload abstractions, scheduling capabilities, networking, storage and policy integrations. Swarm can be simpler for smaller use cases.

---

# 25. Common Interview Traps

## Trap 1 — "Node is a container."

Wrong.

```text
Cluster
 ↓
Node
 ↓
Pod
 ↓
Container
```

---

## Trap 2 — "Pod IP is permanent."

Wrong.

Pod IPs are generally ephemeral.

Use a Service for stable application access.

---

## Trap 3 — "Service creates Pods."

Wrong.

Deployment/ReplicaSet/StatefulSet/etc. manage Pods.

A Service provides networking/discovery/load distribution for selected backends.

---

## Trap 4 — "Ingress is a load balancer."

Not exactly.

Ingress is an API resource defining routing rules. The Ingress Controller or other implementation provides the actual traffic handling.

---

## Trap 5 — "Readiness failure restarts the container."

Normally false.

Readiness controls traffic eligibility.

Liveness is associated with restarting unhealthy containers.

---

## Trap 6 — "Toleration means the Pod will run on the tainted node."

False.

A toleration allows the Pod to be considered for the tainted node; it does not force placement there.

---

## Trap 7 — "StatefulSet means database."

Not exactly.

StatefulSet is a workload controller that provides stable identity/storage/order characteristics useful for stateful applications. It does not turn an application into a database.

---

## Trap 8 — "Kubernetes automatically provides NetworkPolicy enforcement."

Not necessarily.

The network implementation/CNI must support and enforce NetworkPolicy.

---

## Trap 9 — "Running means healthy."

Wrong.

```text
Running
   ≠
Ready
   ≠
Application healthy
```

---

## Trap 10 — "HPA creates more nodes."

No.

HPA changes the **number of Pods**.

Node autoscaling, when configured, is a separate mechanism.

```text
HPA
 ↓
Pods

Node Autoscaler
 ↓
Nodes
```

---

# 26. Practical Debugging Command Sheet

## Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl describe node <node>
```

## Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

## Deployments

```bash
kubectl get deployment
kubectl describe deployment <name>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
```

## ReplicaSets

```bash
kubectl get rs
kubectl describe rs <name>
```

## Services

```bash
kubectl get svc
kubectl describe svc <name>
kubectl get endpoints
kubectl get endpointslices
```

## Networking

```bash
kubectl get networkpolicy
kubectl get pods -o wide
```

## Storage

```bash
kubectl get pv
kubectl get pvc
kubectl get storageclass
kubectl describe pvc <name>
```

## RBAC

```bash
kubectl get role
kubectl get rolebinding
kubectl get clusterrole
kubectl get clusterrolebinding

kubectl auth can-i get pods
kubectl auth can-i get pods \
  --as=system:serviceaccount:app:app-sa
```

## HPA

```bash
kubectl get hpa
kubectl describe hpa <name>
kubectl top pods
kubectl top nodes
```

## Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

Events are often one of the first places to look when diagnosing scheduling, image-pull, probe, and volume problems.

---

# 27. Interview Scenario Framework

When an interviewer gives you a Kubernetes problem, don't randomly run commands.

Use this sequence:

```text
1. Identify object
       ↓
2. Check status
       ↓
3. Describe object
       ↓
4. Read Events
       ↓
5. Check logs
       ↓
6. Check dependencies
       ↓
7. Check networking
       ↓
8. Check node/runtime
       ↓
9. Fix
       ↓
10. Verify
```

Example:

```text
"Service is not working"
        ↓
Service exists?
        ↓
Selector correct?
        ↓
EndpointSlice populated?
        ↓
Pods Ready?
        ↓
Port/targetPort correct?
        ↓
NetworkPolicy?
        ↓
Ingress/LB?
        ↓
Application listening?
```

This demonstrates structured troubleshooting rather than command memorization.

---

# 28. Rapid-Fire Revision Table

| Question | Short interview answer |
|---|---|
| Kubernetes? | Container orchestration platform |
| Pod? | Smallest deployable Kubernetes unit |
| Node? | Machine/VM running Pods |
| API Server? | Central Kubernetes API |
| etcd? | Persistent cluster-state store |
| Scheduler? | Selects nodes for unscheduled Pods |
| Controller? | Reconciles desired and actual state |
| kubelet? | Node agent managing Pods |
| CRI? | Interface between kubelet and container runtime |
| Service? | Stable endpoint for selected Pods |
| ClusterIP? | Internal Service |
| NodePort? | Exposes Service through node port |
| LoadBalancer? | Requests external load-balancer integration |
| Ingress? | HTTP/HTTPS routing resource |
| Ingress Controller? | Implements Ingress behavior |
| CoreDNS? | Cluster DNS |
| CNI? | Container networking interface |
| ReplicaSet? | Maintains desired Pod replicas |
| Deployment? | Manages stateless rollout/lifecycle |
| StatefulSet? | Stable identity/storage workload |
| DaemonSet? | Pod on each eligible node |
| Job? | Run-to-completion workload |
| CronJob? | Scheduled Job |
| ConfigMap? | Non-sensitive configuration |
| Secret? | Sensitive configuration/data |
| PV? | Cluster storage resource |
| PVC? | Storage request |
| StorageClass? | Defines storage provisioning class |
| RBAC? | Authorization model |
| Role? | Namespace-scoped permission definition |
| ClusterRole? | Cluster-level permission definition |
| RoleBinding? | Binds permissions within namespace |
| ClusterRoleBinding? | Cluster-wide binding |
| NetworkPolicy? | Network traffic policy |
| HPA? | Scales Pod replicas from metrics |
| Liveness? | Is container alive? |
| Readiness? | Can Pod receive traffic? |
| Startup? | Has application finished starting? |
| QoS? | Pod resource-quality classification |
| CrashLoopBackOff? | Container repeatedly crashes/restarts |
| ImagePullBackOff? | Image cannot currently be pulled |
| Pending? | Pod cannot yet be scheduled/started |
| NodeNotReady? | Node isn't reporting Ready |
| Kyverno? | Kubernetes-native policy engine |

---

# 29. Final Interview Checklist

Before the interview, make sure you can explain these without looking at notes.

## Architecture

- [ ] Kubernetes
- [ ] Cluster
- [ ] Control plane
- [ ] API Server
- [ ] etcd
- [ ] Scheduler
- [ ] Controller Manager
- [ ] kubelet
- [ ] CRI
- [ ] Container runtime
- [ ] kube-proxy
- [ ] Reconciliation loop

## Pods

- [ ] Pod
- [ ] Pod vs container
- [ ] Pod vs Node
- [ ] Multiple containers
- [ ] Pod IP
- [ ] Pod lifecycle
- [ ] Pod creation flow
- [ ] Self-healing

## Scheduling

- [ ] nodeSelector
- [ ] Node affinity
- [ ] Pod affinity
- [ ] Pod anti-affinity
- [ ] Taints
- [ ] Tolerations
- [ ] NoSchedule
- [ ] PreferNoSchedule
- [ ] NoExecute
- [ ] Scheduling troubleshooting

## Workloads

- [ ] ReplicaSet
- [ ] Deployment
- [ ] StatefulSet
- [ ] DaemonSet
- [ ] Job
- [ ] CronJob
- [ ] Deployment vs StatefulSet
- [ ] Deployment vs DaemonSet
- [ ] Rolling updates
- [ ] Rollback

## Networking

- [ ] Service
- [ ] ClusterIP
- [ ] NodePort
- [ ] LoadBalancer
- [ ] `port`
- [ ] `targetPort`
- [ ] Selectors
- [ ] EndpointSlice
- [ ] kube-proxy
- [ ] CoreDNS
- [ ] CNI
- [ ] Pod networking
- [ ] veth
- [ ] Same-node communication
- [ ] Cross-node communication
- [ ] Overlay networking

## Ingress

- [ ] Ingress
- [ ] Ingress Controller
- [ ] Host-based routing
- [ ] Path-based routing
- [ ] TLS
- [ ] Ingress troubleshooting

## Health

- [ ] Liveness
- [ ] Readiness
- [ ] Startup
- [ ] Running vs Ready
- [ ] Probe troubleshooting

## Resources

- [ ] CPU request
- [ ] CPU limit
- [ ] Memory request
- [ ] Memory limit
- [ ] OOMKilled
- [ ] QoS
- [ ] Guaranteed
- [ ] Burstable
- [ ] BestEffort
- [ ] HPA
- [ ] Metrics

## Configuration

- [ ] ConfigMap
- [ ] Secret
- [ ] Environment injection
- [ ] Volume mounting
- [ ] Secret security

## Storage

- [ ] PV
- [ ] PVC
- [ ] StorageClass
- [ ] Static provisioning
- [ ] Dynamic provisioning
- [ ] `emptyDir`
- [ ] Access modes
- [ ] StatefulSet storage
- [ ] PVC troubleshooting

## Security

- [ ] RBAC
- [ ] Role
- [ ] ClusterRole
- [ ] RoleBinding
- [ ] ClusterRoleBinding
- [ ] ServiceAccount
- [ ] `kubectl auth can-i`
- [ ] NetworkPolicy
- [ ] Default deny
- [ ] CNI enforcement

## Advanced

- [ ] Admission control
- [ ] Mutating admission
- [ ] Validating admission
- [ ] Kyverno
- [ ] Validate
- [ ] Mutate
- [ ] Generate
- [ ] Image verification
- [ ] Docker Swarm vs Kubernetes

## Troubleshooting

- [ ] Pending
- [ ] CrashLoopBackOff
- [ ] ImagePullBackOff
- [ ] NodeNotReady
- [ ] Service without endpoints
- [ ] DNS failure
- [ ] Ingress failure
- [ ] PVC Pending
- [ ] Readiness failure
- [ ] NetworkPolicy blocking traffic

---

# 30. Interview Answer Formula

For most Kubernetes questions, use this structure:

```text
1. Definition
      ↓
2. Why it exists
      ↓
3. How it works
      ↓
4. Small example
      ↓
5. Real DevOps use case
      ↓
6. Important caveat/comparison
```

Example:

> **What is a Service?**

**Definition:** A Service provides a stable network endpoint for a set of Pods.

**Why:** Pod IPs are ephemeral.

**How:** A selector identifies backend Pods and the Service networking dataplane forwards traffic to available endpoints.

**Example:** `ClusterIP` Service exposing an API on port 80 and forwarding to Pod port 8080.

**Use case:** Frontend Pods communicating with backend Pods without knowing individual Pod IPs.

**Caveat:** A Service does not itself create Pods; the workload controller manages them.

---

# End

The most important Kubernetes mental model is:

```text
                    Kubernetes API
                          ↓
                    Desired State
                          ↓
                 Controllers / Scheduler
                          ↓
                       Nodes
                          ↓
                        Pods
                          ↓
                     Containers
                          ↓
              Networking / Storage / Services
                          ↓
                    Actual State
                          ↓
                 Reconciliation Loop
                          ↓
                 Desired State restored
```

For interview success, focus especially on **architecture, Pod lifecycle, scheduling, Deployments, Services, DNS, networking/CNI, probes, storage, RBAC, NetworkPolicy, and troubleshooting scenarios**.

A strong answer should explain not only **what** a Kubernetes object is, but also **why it exists, what happens internally, how it interacts with other components, and how you would troubleshoot it in production**.
