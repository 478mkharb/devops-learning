# Kubernetes Interview Questions

This README is organized in ascending conceptual order: **Kubernetes → Kubernetes vs Docker → fundamentals → architecture → Pods → workloads → scheduling → networking → health/resources → storage → security → troubleshooting**.

## Contents

- [1. Kubernetes Fundamentals](#1-kubernetes-fundamentals)
- [2. Kubernetes Architecture](#2-kubernetes-architecture)
- [3. Pods](#3-pods)
- [4. Workloads and Controllers](#4-workloads-and-controllers)
- [5. Services and Networking](#5-services-and-networking)
- [6. Probes, Resources and Application Health](#6-probes-resources-and-application-health)
- [7. ConfigMaps, Secrets and Storage](#7-configmaps-secrets-and-storage)
- [8. RBAC and Kubernetes Security](#8-rbac-and-kubernetes-security)
- [9. Kubernetes Troubleshooting](#9-kubernetes-troubleshooting)
- [Quick Reference Tables](#kubernetes-resource-and-workload-quick-reference)
- [Practical Examples](#practical-examples)
- [Interview Answer Pattern](#interview-answer-pattern)

## 1. Kubernetes Fundamentals

### 1. What is Kubernetes?

Kubernetes is an open-source platform for managing containerized workloads and services. It provides scheduling, service discovery, scaling, controlled deployments, health management, storage integration and declarative reconciliation across a cluster.

| Kubernetes capability | What it provides |
|---|---|
| Scheduling | Places Pods on suitable nodes |
| Controllers | Reconcile desired and actual state |
| Services | Stable service discovery and networking |
| Workload management | Deployments, StatefulSets, Jobs, CronJobs, etc. |
| Storage | PV, PVC and StorageClass abstractions |
| Security | Authentication, authorization and policies |

**Example:** Example: `replicas: 3` declares the desired number of application replicas; controllers work to maintain that state.

---

### 2. What is the difference between Kubernetes and Docker?

Docker is commonly used to build, package and run containers. Kubernetes is an orchestration platform that manages containerized workloads across a cluster. They solve related but different problems.

| Docker | Kubernetes |
|---|---|
| Builds/packages/runs containers | Orchestrates containerized workloads |
| Commonly used for local container workflows | Manages workloads across a cluster |
| Provides container tooling/runtime ecosystem | Provides API server, scheduler, controllers and workload APIs |
| Does not provide Kubernetes-style reconciliation | Continuously reconciles desired and actual state |

**Example:** Example: Docker can build `myapp:1.0`; Kubernetes can deploy that image through a Deployment.

---

### 3. What is the difference between `podAffinity` and `podAntiAffinity`?


Pod affinity tries to place Pods together according to matching labels and topology rules.

Pod anti-affinity tries to keep matching Pods apart.

Anti-affinity is commonly used for high availability, for example to prevent multiple replicas of the same application from being scheduled onto the same node.

---

| Layer | Main responsibility |
|---|---|
| Control plane | Manages cluster state |
| Worker node | Runs Pods |
| API Server | Exposes the Kubernetes API |
| etcd | Stores persistent cluster state |
| Scheduler | Selects nodes for unscheduled Pods |
| Controllers | Reconcile resources |

**Example:** Example architecture: `kubectl → API Server → control plane → worker node → kubelet → runtime → Pod`.

---

### 4. What is the difference between taints/tolerations and node affinity?


A taint is applied to a node and says that Pods should not be scheduled there unless they tolerate the taint.

A toleration is applied to a Pod and allows it to be considered for a tainted node.

Node affinity is different: it is a Pod-side rule that expresses which nodes the Pod wants or requires.

An important interview point is that a toleration does **not** force a Pod onto a node. It only allows the Pod to tolerate the taint.

---

| Node | Pod |
|---|---|
| Worker machine | Smallest deployable compute unit |
| Provides CPU/memory/networking | Consumes node resources |
| Can host many Pods | Contains one or more containers |
| Part of cluster infrastructure | Represents an application workload unit |

**Example:** Example: a worker Node can host multiple frontend, API and worker Pods until resource/scheduling constraints prevent placement.

---

### 5. What is the difference between a node label and a node taint?


A label is metadata used for selection and grouping.

A taint is a scheduling restriction applied to a node.

Labels answer: **which Pods should prefer or require this node?**

Taints answer: **which Pods are allowed to use this node?**

---

| Node label | Node taint |
|---|---|
| Metadata used for selection/grouping | Scheduling restriction |
| Helps a Pod choose a node | Repels Pods unless tolerated |
| Used with nodeSelector/affinity | Used with tolerations |

**Example:** Example: `kubectl label node node1 workload=database` adds a selection label; a taint can then repel unrelated workloads.

---

### 6. What is the difference between `NoSchedule`, `PreferNoSchedule`, and `NoExecute`?


`NoSchedule` prevents new Pods from being scheduled unless they tolerate the taint.

`PreferNoSchedule` is a soft preference to avoid scheduling there.

`NoExecute` prevents new scheduling and can also evict existing Pods that do not have the required toleration.

---

| Taint effect | New Pod without toleration | Existing Pod |
|---|---|---|
| `NoSchedule` | Not scheduled | Normally remains |
| `PreferNoSchedule` | Scheduler prefers to avoid | Remains |
| `NoExecute` | Not scheduled | Can be evicted |

**Example:** Example: `workload=database:NoSchedule` blocks new Pods without a matching toleration.

---

### 7. Does a toleration force a Pod to run on a particular node?


No.

A toleration only makes a Pod eligible to run on a tainted node. It does not select that node.

If I need to target a particular class of nodes, I would combine tolerations with node labels and node affinity or nodeSelector.

---

---

### 8. What is the difference between `requiredDuringSchedulingIgnoredDuringExecution` and `preferredDuringSchedulingIgnoredDuringExecution`?


`requiredDuringSchedulingIgnoredDuringExecution` is a hard requirement. If no node satisfies it, the Pod cannot be scheduled.

`preferredDuringSchedulingIgnoredDuringExecution` is a soft preference. The scheduler tries to satisfy it but can choose another node if necessary.

---

| Affinity rule | Meaning |
|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | Hard requirement |
| `preferredDuringSchedulingIgnoredDuringExecution` | Soft preference |
| `IgnoredDuringExecution` | Existing Pod is not automatically evicted when labels later change |

**Example:** Example: required affinity can force a Pod onto nodes labeled `workload=database`; preferred affinity only expresses a preference.

---

### 9. What does `IgnoredDuringExecution` mean in node affinity?


It means the rule is considered during scheduling, but if node labels later change and the Pod no longer matches the rule, Kubernetes does not automatically evict that running Pod because of the affinity rule.

---

---

### 10. Why would a Pod remain in `Pending` because of scheduling?


Typical reasons include insufficient CPU or memory, node affinity rules that match no nodes, taints without matching tolerations, node selectors that match no nodes, resource constraints, or scheduling constraints such as volume topology.

I would start with:

```bash
kubectl describe pod <pod>
kubectl get nodes --show-labels
kubectl describe nodes
```

Then I would inspect the scheduler events for the actual reason.

---

**Example:** Troubleshooting example: `kubectl describe pod <pod>` and Events can reveal insufficient resources, affinity mismatch or an untolerated taint.

---

### 11. How would you dedicate nodes to a particular workload?


I would normally use a combination of node labels, taints/tolerations, and node affinity.

For example, I could label nodes as:

```bash
kubectl label node node1 workload=database
```

Then taint them so general workloads stay away, give database Pods the corresponding toleration, and use node affinity to ensure the database Pods actually select those nodes.

---

---

### 12. What is the role of the Kubernetes scheduler?


The scheduler watches for newly created Pods that do not yet have a node assignment.

It evaluates available nodes against scheduling constraints such as resource requests, affinity, taints, topology and other policies, then selects an appropriate node and binds the Pod to it.

---

**Example:** Mental model: scheduler = **where** the Pod runs; kubelet = **make the assigned Pod run there**.

---

### 13. Does the scheduler start containers?


No.

The scheduler is responsible for selecting a node and assigning the Pod to it.

After that, the kubelet on the selected node is responsible for making sure the Pod's containers are created and running.

---

---

### 14. What is the difference between scheduling and kubelet responsibility?


The scheduler decides **where** the Pod should run.

The kubelet on that node is responsible for ensuring the Pod actually runs there, including interacting with the container runtime and reporting Pod status back to the control plane.

---

---

### 15. How can you prevent two replicas from running on the same node?


I can use Pod anti-affinity or topology spread constraints.

For example, Pod anti-affinity can require replicas with the same application label to be placed on different nodes.

---

## 2. Pods and Workloads

**Example:** Example: Pod anti-affinity can keep replicas apart across nodes; topology spread constraints can distribute replicas across topology domains.

---

## 2. Kubernetes Architecture

### 16. What are the major Kubernetes control-plane components?


The major components are:

- API Server
- etcd
- Scheduler
- Controller Manager

Cloud environments may also have a cloud-controller-manager.

The kubelet and container runtime run on worker nodes rather than being control-plane components.

---

| Control plane | Worker node |
|---|---|
| kube-apiserver | kubelet |
| etcd | container runtime |
| kube-scheduler | kube-proxy where used |
| kube-controller-manager | Pods |

**Example:** A common architecture is a control plane managing one or more worker nodes; production control planes are commonly deployed for high availability.

---

### 17. What is the role of the API Server?


The API Server is the central API endpoint of the Kubernetes control plane.

`kubectl`, controllers, operators and other clients communicate with Kubernetes through the API Server.

It handles authentication, authorization, admission and API operations.

---

| API Server stage | Purpose |
|---|---|
| Authentication | Identify the caller |
| Authorization | Decide whether the caller is allowed |
| Admission | Validate/mutate requests |
| API operation | Read/create/update/delete resources |

**Example:** Request path: `kubectl → API Server → authentication → authorization → admission → API operation`.

---

### 18. What is etcd?


etcd is the strongly consistent distributed key-value store used by Kubernetes to persist cluster state.

It stores important Kubernetes objects and configuration state.

---

**Example:** Example: etcd persists Kubernetes API state, including objects such as Deployments, Services and Pods.

---

### 19. What does the kubelet do?


The kubelet runs on each worker node.

It receives Pod assignments, works with the container runtime to create and manage containers, performs health checks, mounts volumes and reports node/Pod status back to the API Server.

---

| Component | Main responsibility |
|---|---|
| Scheduler | Decides where an unscheduled Pod runs |
| Kubelet | Ensures assigned Pods run on its node |
| Container runtime | Creates/runs containers |

**Example:** Example: scheduler selects `worker-2`; kubelet on `worker-2` works with the runtime to create the Pod.

---

### 20. What is the role of the container runtime?


The container runtime is responsible for actually creating and running containers according to the Kubernetes CRI interface.

Examples in modern Kubernetes environments include containerd and CRI-O.

---

| Runtime | Role |
|---|---|
| containerd | CRI-compatible container runtime commonly used with Kubernetes |
| CRI-O | Kubernetes-focused CRI implementation |

**Example:** Examples include containerd and CRI-O.

---

### 21. What is the difference between kubelet and kube-proxy?


Kubelet manages Pods and containers on the node.

kube-proxy historically manages Service-related network forwarding rules on nodes. Modern Kubernetes networking can also use eBPF-based implementations that reduce or replace the traditional kube-proxy datapath.

---

| kubelet | kube-proxy |
|---|---|
| Manages Pods/containers on a node | Implements Service networking datapath where used |
| Reports node/Pod status | Maintains relevant node networking rules |

**Example:** Example: kubelet manages Pod lifecycle while Service networking may be implemented through kube-proxy or another networking datapath.

---

### 22. What is the role of the controller manager?


It runs controller processes that continuously reconcile desired state with actual state.

Examples include controllers responsible for Deployments, ReplicaSets, Nodes, Jobs and other Kubernetes resources.

---

**Example:** Example controllers manage Deployments/ReplicaSets, Jobs and Nodes.

---

### 23. Why is Kubernetes called declarative?


Instead of telling Kubernetes every individual step to perform, we declare the desired end state.

For example:

```yaml
replicas: 3
```

means we want three replicas.

Kubernetes controllers continuously work to make the actual state converge toward that desired state.

---

## 8. Real Interview Troubleshooting Scenarios

**Example:** You declare `replicas: 3` instead of scripting the creation of three individual containers.

---

### 24. What happens when you run `kubectl apply -f deployment.yaml`?


At a high level:

```text
kubectl
  ↓
API Server
  ↓
Authentication / Authorization / Admission
  ↓
Object stored in etcd
  ↓
Deployment Controller
  ↓
ReplicaSet
  ↓
Pod
  ↓
Scheduler
  ↓
Node
  ↓
Kubelet
  ↓
Container Runtime
  ↓
Container
```

Controllers continuously reconcile the desired state with the actual state.

---

**Example:** Example: `kubectl apply` submits desired configuration through the API Server; controllers then reconcile the cluster.

---

### 25. What is the difference between desired state and actual state?


Desired state is what the Kubernetes API objects specify should exist.

Actual state is what currently exists in the cluster.

Controllers continuously compare the two and take actions to reduce the difference.

This reconciliation model is fundamental to Kubernetes.

---

**Example:** Example: desired replicas = 3 but only 2 exist → the controller creates another Pod.

---

## 3. Pods

### 26. Why does Kubernetes use Pods instead of scheduling containers directly?


The Pod is Kubernetes' smallest deployable unit.

It provides a shared execution context for one or more containers, including networking and potentially shared storage.

This allows tightly coupled containers such as an application container and a sidecar to share the same network namespace and lifecycle boundary.

---

**Example:** Example: an application container and a sidecar can share `localhost` because containers in one Pod share a network namespace.

---

### 27. Why is one container per Pod common if Pods can contain multiple containers?


A Pod can contain multiple containers, but containers should normally be placed together only when they need to share lifecycle, networking, or storage.

For independent application components, separate Pods are usually better because they can scale and restart independently.

So one-container Pods are common, but Kubernetes does not require it.

---

**Example:** Use multiple containers in one Pod when they are tightly coupled; use separate Pods for independently scalable components.

---

### 28. What is the difference between a Pod and a container?


A container is the process/application runtime unit created by the container runtime.

A Pod is the Kubernetes scheduling and deployment unit that can contain one or more containers.

Kubernetes schedules the Pod, not an individual container.

---

| Pod | Container |
|---|---|
| Kubernetes scheduling/deployment unit | Runtime unit |
| Can contain one or more containers | Individual container execution unit |
| Containers in a Pod share Pod networking | Has its own container isolation context |

**Example:** Example: Kubernetes schedules a Pod containing an nginx container, not nginx as a separate scheduling object.

---

### 29. What is the difference between a Pod and a Node?


A Node is a worker machine, physical or virtual, that provides CPU, memory, networking and storage.

A Pod is a workload unit scheduled onto a Node.

A Node can run multiple Pods, depending on its resources and scheduling constraints.

---

**Example:** Example: `worker-1` can run several Pods simultaneously.

---

### 30. What happens when a Pod is deleted?


If it is managed by a controller such as a Deployment or ReplicaSet, the controller detects that the desired replica count is no longer satisfied and creates a replacement Pod.

If the Pod was standalone, Kubernetes does not automatically recreate it.

---

**Example:** A Deployment-managed Pod is replaced to restore the desired replica count; a standalone Pod is not automatically recreated by a Deployment controller.

---

### 31. Why are Pod IPs considered ephemeral?


Pods are replaceable workload instances.

When a Pod is recreated, the new Pod can receive a different IP address. Therefore applications should not normally depend on a Pod IP directly.

Services provide stable discovery for workloads.

---

**Example:** Example: replacing a Pod can result in a new IP, so clients normally use a Service.

---

## 4. Workloads and Controllers

### 32. What is the difference between a Deployment and a ReplicaSet?


A ReplicaSet maintains the desired number of matching Pods.

A Deployment manages ReplicaSets and adds higher-level capabilities such as rolling updates, rollbacks and revision history.

In normal application deployments, I create a Deployment rather than managing ReplicaSets directly.

---

| Deployment | ReplicaSet |
|---|---|
| Manages rollouts and ReplicaSets | Maintains desired count of matching Pods |
| Supports rollout/rollback | Lower-level replica controller |

**Example:** Example: `Deployment → ReplicaSet → Pods`.

---

### 33. What happens when you update the image of a Deployment?


Changing the Pod template creates a new ReplicaSet.

The Deployment then performs a rolling update by gradually increasing the new ReplicaSet and reducing the old ReplicaSet according to its update strategy.

---

**Example:** Changing the image in a Deployment Pod template creates a new ReplicaSet and starts a rollout.

---

### 34. What is a ReplicaSet's role during a rolling deployment?


The Deployment creates and manages ReplicaSets.

During a rolling update, the new ReplicaSet scales up while the old ReplicaSet scales down, allowing controlled replacement of Pods.

---

**Example:** During a rolling update, old and new ReplicaSets can coexist temporarily.

---

### 35. How would you rollback a Deployment?


I would first inspect rollout history:

```bash
kubectl rollout history deployment/<name>
```

Then rollback:

```bash
kubectl rollout undo deployment/<name>
```

After that I would verify:

```bash
kubectl rollout status deployment/<name>
```

and check the application and Pods.

---

## 3. Services and Networking

**Example:** Commands: `kubectl rollout history deployment/<name>` and `kubectl rollout undo deployment/<name>`.

---

### 36. What is the difference between Deployment and StatefulSet?


Deployment is normally used for stateless applications where replicas are interchangeable.

StatefulSet is intended for workloads that need stable Pod identity, stable network identity, or persistent storage association.

Examples include databases and clustered systems that require predictable identities.

---

| Deployment | StatefulSet |
|---|---|
| Usually interchangeable replicas | Stable identity/storage association |
| Common for stateless services | Common for stateful workloads |

**Example:** Example: stateless API = Deployment; stateful clustered workload requiring stable identity = StatefulSet.

---

### 37. Why does a StatefulSet give Pods stable identities?


StatefulSet Pods have predictable ordinal identities such as:

```text
database-0
database-1
database-2
```

Those identities remain associated with the corresponding Pod ordinal across normal Pod recreation.

This is useful for distributed systems that need stable member identities.

---

**Example:** Example identities: `database-0`, `database-1`, `database-2`.

---

### 38. What is the difference between StatefulSet and DaemonSet?


StatefulSet manages an ordered or identity-aware set of replicas.

DaemonSet ensures a Pod runs on eligible nodes, commonly one Pod per node.

DaemonSets are commonly used for node-level agents such as log collectors and monitoring agents.

---

| StatefulSet | DaemonSet |
|---|---|
| Identity-aware replicas | Pod on each eligible node |
| Often stateful applications | Common for node agents |

**Example:** Example: a node-level log collector is commonly a DaemonSet.

---

### 39. What is the difference between DaemonSet and Deployment?


A Deployment controls a desired number of replicas independent of node count.

A DaemonSet schedules a Pod on every eligible node, or on every node matching its constraints.

---

**Example:** Deployment replica count is independent of node count; DaemonSet targets eligible nodes.

---

### 40. What happens when a new node joins a cluster with a DaemonSet?


The DaemonSet controller sees that the new eligible node does not have the DaemonSet Pod and schedules one there.

This is why DaemonSets are useful for node-level agents.

---

**Example:** When a new eligible node joins, the DaemonSet controller creates its Pod there.

---

### 41. What is a Job?


A Job is used for a finite task that should run to completion.

It creates Pods and tracks successful completion rather than maintaining an indefinitely running application.

---

| Deployment | Job |
|---|---|
| Long-running application | Finite task |
| Maintains desired replicas | Tracks successful completion |

**Example:** Example: a database migration that must finish can be a Job.

---

### 42. What is the difference between Job and Deployment?


A Deployment is intended for continuously running applications.

A Job is intended for finite work that eventually completes.

For example, an API server would normally use a Deployment, while a database migration could use a Job.

---

**Example:** Example: API server = Deployment; schema migration = Job.

---

### 43. What is a CronJob?


A CronJob creates Jobs according to a schedule expressed using cron syntax.

It is useful for recurring batch tasks such as reports, cleanup tasks, backups, or periodic maintenance.

---

**Example:** Example: a nightly cleanup task can be a CronJob.

---

### 44. What is the difference between Job and CronJob?


A Job represents one finite execution.

A CronJob is a scheduler for creating Jobs repeatedly according to a schedule.

---

| Job | CronJob |
|---|---|
| Finite execution | Creates Jobs according to a schedule |
| Runs a task | Repeats task creation |

**Example:** Mental model: Job = finite task; CronJob = scheduled creation of Jobs.

---

### 45. What is the difference between scaling a Deployment and scaling a StatefulSet?


Both can be scaled by changing replicas, but StatefulSet scaling preserves stable identities and associated storage relationships.

Deployment replicas are generally interchangeable, while StatefulSet replicas have stable identities.

---

**Example:** Scaling a StatefulSet preserves ordinal identities such as `db-0` and adds/removes higher ordinals.

---

## 5. Services and Networking

### 46. Why do we need a Service when Pods already have IP addresses?


Pod IPs are not stable because Pods can be recreated.

A Service provides a stable virtual IP and DNS name and selects backend Pods using labels.

Applications therefore communicate with the Service rather than tracking individual Pod IPs.

---

| Pod IP | Service |
|---|---|
| Belongs to one Pod | Represents a logical backend group |
| Can change when Pod is recreated | Stable virtual endpoint |
| Clients should normally avoid direct dependency | Provides service discovery |

**Example:** Example: `10.0.2.17` may disappear with a Pod; clients continue using the Service DNS name.

---

### 47. What is the difference between ClusterIP, NodePort and LoadBalancer?


ClusterIP exposes the Service internally within the cluster and is the default.

NodePort exposes the Service through a port on each node.

LoadBalancer normally integrates with a cloud provider to provision an external load balancer.

A typical external flow is:

```text
Client
  |
Load Balancer
  |
Service
  |
Pods
```

---

| Service type | Typical use |
|---|---|
| ClusterIP | Internal cluster access |
| NodePort | Node-level exposure |
| LoadBalancer | External load-balancer integration |
| Headless | Direct backend/Pod discovery |

**Example:** Use ClusterIP for internal access, NodePort for node-level exposure and LoadBalancer for provider-integrated external exposure.

---

### 48. What is the difference between Service port and targetPort?


`port` is the port exposed by the Service.

`targetPort` is the port on the selected Pod/container where traffic is forwarded.

For example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

means clients connect to Service port 80 and traffic is forwarded to port 8080 on the backend Pods.

---

| Field | Meaning |
|---|---|
| `port` | Port exposed by the Service |
| `targetPort` | Port on the selected backend Pod |

**Example:** Example: client → Service port 80 → backend Pod port 8080.

---

### 49. What is `nodePort`?


`nodePort` is a port exposed on each eligible node for a NodePort Service.

Traffic arriving at a node's IP and that port can be forwarded through the Service to its backend Pods.

---

**Example:** A NodePort exposes a Service through a port on each eligible node.

---

### 50. What happens if a Service has no endpoints?


The Service exists, but it has no backend Pods that currently match its selector and qualify as endpoints.

I would check:

```bash
kubectl get svc <service>
kubectl get endpoints <service>
kubectl get endpointslices
kubectl get pods --show-labels
```

Then compare the Service selector with the Pod labels and check readiness.

---

**Example:** Use `kubectl get endpointslices` to check whether a Service has backend endpoints.

---

### 51. A Service selector looks correct, but it still has no endpoints. What do you check?


I would check whether:

1. Pod labels exactly match the Service selector.
2. Pods are in the same namespace as the Service.
3. Pods are Running and Ready.
4. The Service `targetPort` matches the application's listening port.
5. EndpointSlices contain the expected addresses.

Commands:

```bash
kubectl describe svc <service>
kubectl get pods --show-labels
kubectl get endpoints <service>
kubectl get endpointslices
```

---

**Example:** Check namespace, Service selector, Pod labels, readiness, `targetPort` and EndpointSlices.

---

### 52. What is the difference between Endpoints and EndpointSlice?


Endpoints is the older representation of Service backend addresses.

EndpointSlice is the newer, more scalable mechanism that divides endpoints across multiple objects instead of maintaining one potentially large object.

---

| Endpoints | EndpointSlice |
|---|---|
| Older representation | Newer scalable representation |
| Can become one large object | Backend endpoints can be split across slices |

**Example:** EndpointSlice distributes endpoint information across multiple objects for scalability.

---

### 53. How does Kubernetes Service discovery work?


Kubernetes DNS creates DNS records for Services.

A client can resolve a Service by name, for example:

```text
my-service
```

or, depending on namespace:

```text
my-service.my-namespace
```

The DNS name resolves to the Service's stable virtual IP, after which Kubernetes networking forwards traffic to backend endpoints.

---

**Example:** Example: `api.default.svc` can be used for cluster service discovery.

---

### 54. What is the difference between a Service IP and a Pod IP?


A Pod IP belongs to an individual Pod and can change when the Pod is recreated.

A Service IP is a stable virtual address representing a logical group of Pods.

---

**Example:** A Pod IP can change when the Pod is recreated; a Service provides the stable logical endpoint.

---

### 55. What is a headless Service?


A headless Service is created with:

```yaml
clusterIP: None
```

It does not allocate a normal virtual ClusterIP. DNS can instead return the addresses of the backing Pods.

It is commonly used with StatefulSets and systems where clients need to discover individual Pod identities.

---

**Example:** A headless Service uses `clusterIP: None` and is useful when clients need direct backend/Pod discovery.

---

### 56. What is Ingress?


Ingress is an API resource that defines HTTP/HTTPS routing rules into Services.

An Ingress controller implements those rules using an actual proxy/load-balancing implementation.

---

| Ingress | LoadBalancer Service |
|---|---|
| HTTP/HTTPS routing across Services | Exposes a Service through external LB integration |
| Supports host/path routing | Simpler external Service exposure |
| Requires an implementation/controller | Depends on provider integration |

**Example:** Example: `example.com/api → api-service` and `example.com/web → frontend-service`.

---

### 57. What is the difference between Ingress and Ingress Controller?


Ingress is the Kubernetes API object containing routing configuration.

The Ingress Controller is the component that watches those resources and actually handles traffic according to the configured rules.

---

| Ingress | Ingress Controller |
|---|---|
| API routing configuration | Implements the routing/data plane |

**Example:** An Ingress object is configuration; the Ingress Controller implements the routing.

---

### 58. Can an Ingress work without an Ingress Controller?


Creating an Ingress object alone does not provide the data-plane implementation.

An Ingress Controller is required to actually process the rules and route traffic.

---

**Example:** Creating an Ingress resource without an implementation does not itself process application traffic.

---

### 59. What is the difference between Ingress and LoadBalancer Service?


A LoadBalancer Service generally exposes a Service through an external load balancer.

Ingress provides Layer 7 HTTP/HTTPS routing, allowing multiple hosts or URL paths to be routed to different Services.

For example:

```text
example.com/api  → api-service
example.com/web  → frontend-service
```

---

**Example:** Use LoadBalancer Service for straightforward external exposure; use Ingress when consolidated HTTP/HTTPS routing is required.

---

### 60. How would you troubleshoot a Service that is reachable but returns connection errors?


I would trace the path:

```text
Client → Ingress/LB → Service → EndpointSlice → Pod → Application
```

Then check:

```bash
kubectl get ingress
kubectl get svc
kubectl get endpointslices
kubectl get pods -o wide
kubectl describe svc <service>
kubectl logs <pod>
```

I would also verify ports, selectors, NetworkPolicies, DNS and whether the application is actually listening on the expected interface and port.

---

## 4. Probes, Resources and Application Health

**Example:** Trace traffic: client → DNS → Ingress/LB → Service → EndpointSlice → Pod → application.

---

## 6. Probes, Resources and Application Health

### 61. What is the difference between readiness, liveness and startup probes?


Readiness determines whether a Pod should receive traffic.

Liveness determines whether the container is still healthy enough to continue running; repeated failures can cause a restart.

Startup is designed for applications that take a long time to initialize and prevents liveness checks from killing them during startup.

---

| Probe | Main question |
|---|---|
| Startup | Has startup completed? |
| Readiness | Should this container receive traffic? |
| Liveness | Is this container unhealthy enough to restart? |

**Example:** Startup protects slow initialization, readiness controls traffic, liveness can trigger restarts.

---

### 62. If readiness fails, is the container restarted?


Normally, no.

A readiness failure causes the Pod to be treated as not ready and removed from Service endpoints.

A liveness failure can cause the container to be restarted.

---

**Example:** Readiness failure normally removes a Pod from matching Service endpoints rather than restarting it.

---

### 63. Why would you use a startup probe?


For applications with slow or unpredictable startup times.

It allows the application to finish initialization before liveness probing becomes active.

---

**Example:** Example: a slow-starting Java application can use startup probing before liveness checks become active.

---

### 64. What is the difference between CPU request and CPU limit?


CPU request is the amount Kubernetes uses for scheduling and resource accounting.

CPU limit is the maximum CPU the container is allowed to consume.

CPU is compressible, so CPU throttling can occur when a container reaches its CPU limit.

---

| Request | Limit |
|---|---|
| Used for scheduling/resource accounting | Runtime consumption ceiling |
| Describes expected resource need | Restricts maximum configured usage |

**Example:** Example: request `500m` CPU and limit `1` CPU; the request influences scheduling while the limit constrains usage.

---

### 65. What is the difference between memory request and memory limit?


Memory request is used when the scheduler decides whether a node has enough allocatable memory.

Memory limit restricts the container's memory consumption.

Unlike CPU, memory is not compressible. If a container exceeds its memory limit, it can be terminated with an OOM condition.

---

| Resource | Typical enforcement behavior |
|---|---|
| CPU | Can be throttled |
| Memory | Can result in OOM termination when limit is exceeded |

**Example:** CPU can be throttled; exceeding a memory limit can result in an OOM kill.

---

### 66. Why can a Pod be `OOMKilled` even if the node has free memory?


The container may have exceeded its configured memory limit.

Container-level limits can cause an OOM kill even when the node still has available memory.

I would inspect:

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o yaml
```

and review resource usage and application memory behavior.

---

**Example:** Inspect `kubectl describe pod` for `OOMKilled` and review the container memory limit.

---

### 67. What happens if a Pod has no resource requests?


The scheduler has less information about the Pod's expected resource consumption.

This can lead to poor scheduling decisions and resource contention.

In production, I prefer defining appropriate requests and limits based on observed workload behavior.

---

**Example:** Without requests, the scheduler has less declared information about expected resource consumption.

---

### 68. What is QoS in Kubernetes?


Kubernetes assigns Pods a Quality of Service class based on their CPU and memory requests and limits.

The main classes are:

- Guaranteed
- Burstable
- BestEffort

QoS affects behavior during resource pressure, especially eviction decisions.

---

| QoS class | General condition |
|---|---|
| Guaranteed | Containers satisfy Guaranteed CPU/memory request/limit criteria |
| Burstable | Requests/limits exist but Guaranteed criteria are not met |
| BestEffort | No CPU/memory requests or limits |

**Example:** QoS classification becomes particularly important during resource pressure.

---

### 69. What is the difference between Guaranteed and Burstable QoS?


A Pod is generally Guaranteed when every container has CPU and memory requests and limits set, with requests equal to limits for those resources.

Burstable Pods have requests and/or limits configured but do not satisfy the Guaranteed criteria.

---

**Example:** If containers satisfy the required CPU/memory request-limit equality criteria, the Pod can be Guaranteed.

---

### 70. What is `CrashLoopBackOff`?


It means a container repeatedly starts and exits, so Kubernetes applies an increasing restart backoff delay.

It is a symptom rather than a root cause.

I would inspect:

```bash
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

and investigate application errors, configuration, missing dependencies, permissions, probes and resource limits.

---

**Example:** CrashLoopBackOff is a symptom; inspect previous logs, events, probes, configuration and dependencies.

---

### 71. What is the difference between `CrashLoopBackOff` and `ImagePullBackOff`?


`CrashLoopBackOff` means the container starts but repeatedly exits and Kubernetes backs off restarting it.

`ImagePullBackOff` means Kubernetes cannot successfully pull the container image and is backing off subsequent pull attempts.

---

**Example:** Bad application startup can cause CrashLoopBackOff; an inaccessible/nonexistent image can cause ImagePullBackOff.

---

### 72. How would you troubleshoot `ImagePullBackOff`?


I would inspect:

```bash
kubectl describe pod <pod>
```

especially Events.

Then I would verify:

- Image name and tag
- Registry connectivity
- ImagePullSecrets
- Registry authentication
- Node network connectivity
- Whether the image actually exists

---

## 5. ConfigMaps, Secrets and Storage

**Example:** Check image/tag, registry connectivity, credentials/ImagePullSecrets, image existence and node networking.

---

## 7. ConfigMaps, Secrets and Storage

### 73. What is the difference between ConfigMap and Secret?


ConfigMap stores non-sensitive configuration.

Secret is intended for sensitive data such as credentials, tokens and certificates.

A Kubernetes Secret is not automatically encrypted merely because it is called a Secret; values are commonly base64 encoded, while encryption at rest must be configured separately.

---

| ConfigMap | Secret |
|---|---|
| Non-sensitive configuration | Sensitive configuration |
| Not an encryption mechanism | Base64 encoding alone is not encryption |
| Used for application settings | Requires appropriate access/storage protection |

**Example:** Example: `APP_ENV=prod` can be ConfigMap data; a database password belongs in Secret data.

---

### 74. Are Kubernetes Secrets encrypted by default?


Base64 encoding is not encryption.

Whether Secret data is encrypted at rest depends on the cluster's API server/storage encryption configuration.

For sensitive production environments, I would verify encryption-at-rest configuration and access controls.

---

**Example:** Base64 encoding is not encryption; verify encryption-at-rest configuration for sensitive Secret storage.

---

### 75. What is the difference between environment-variable and volume-based ConfigMap usage?


With environment variables, configuration is injected into the container environment.

With a volume, Kubernetes mounts configuration as files.

Volume-based configuration is useful when applications naturally consume configuration files.

---

**Example:** Use environment variables for simple values and volume-mounted ConfigMaps when the application expects configuration files.

---

### 76. What is a PersistentVolume?


A PersistentVolume is cluster storage represented as a Kubernetes resource.

It abstracts the underlying storage implementation from the application Pod.

---

| PV | PVC |
|---|---|
| Provisioned storage resource | Workload request for storage |
| Cluster storage object | Can bind to a PV or trigger dynamic provisioning |

**Example:** A cloud block volume can be represented by a PV and consumed through a PVC.

---

### 77. What is a PersistentVolumeClaim?


A PVC is a request for storage made by a workload.

The PVC can bind to a suitable PersistentVolume or trigger dynamic provisioning through a StorageClass.

---

**Example:** A Pod references a PVC rather than needing to know the underlying storage implementation.

---

### 78. What is the difference between PV and PVC?


PV represents the provisioned storage resource.

PVC represents a user's/workload's request for storage.

A useful mental model is:

```text
Pod → PVC → PV → Storage backend
```

---

**Example:** Mental model: `Pod → PVC → PV → storage backend`.

---

### 79. What is a StorageClass?


A StorageClass defines a class of storage and provides the information Kubernetes needs for dynamic provisioning.

For example, a cloud StorageClass can dynamically create block volumes when a PVC is requested.

---

| Static provisioning | Dynamic provisioning |
|---|---|
| Administrator creates/provides PV | PVC can trigger storage creation |
| More manual | StorageClass automates provisioning |

**Example:** A StorageClass can dynamically provision storage when a PVC requests it.

---

### 80. What is dynamic provisioning?


Dynamic provisioning automatically creates storage when a PVC requests it, instead of requiring an administrator to manually create a PV beforehand.

---

**Example:** Dynamic provisioning avoids manually pre-creating every PV.

---

### 81. What is the difference between `emptyDir` and a PersistentVolume?


`emptyDir` is temporary Pod-local storage. It exists while the Pod exists and is lost when the Pod is removed from the node.

A PersistentVolume provides storage intended to survive Pod recreation according to the storage backend and reclaim policy.

---

**Example:** `emptyDir` is suitable for temporary scratch data that can be lost with Pod removal.

---

### 82. What happens to a PVC when its Pod is deleted?


Deleting the Pod normally does not delete the PVC.

The PVC remains so that a replacement Pod can mount the same claim, assuming the workload is configured to use it.

---

**Example:** Deleting a Pod normally does not delete its PVC.

---

### 83. What is a StatefulSet volume claim template?


A StatefulSet can use `volumeClaimTemplates` to create a separate PVC for each StatefulSet Pod.

This allows identities such as `database-0` and `database-1` to have their own persistent storage.

---

**Example:** StatefulSet `volumeClaimTemplates` can provide separate persistent claims for individual Pods.

---

### 84. What does `ReadWriteOnce` mean?


It generally means the volume can be mounted read-write by a single node at a time.

The exact semantics depend on the storage implementation.

---

**Example:** ReadWriteOnce generally permits read-write mounting by a single node at a time; exact semantics depend on the storage implementation.

---

### 85. Why would a Pod be stuck in `Pending` because of storage?


Possible reasons include an unbound PVC, no matching StorageClass, insufficient storage capacity, topology constraints, access-mode incompatibility, or a dynamic provisioning failure.

I would check:

```bash
kubectl get pvc
kubectl describe pvc <pvc>
kubectl get pv
kubectl get storageclass
```

---

## 6. RBAC and Security

**Example:** Check PVC events, StorageClass, PV availability, access mode and topology when storage prevents scheduling.

---

## 8. RBAC and Kubernetes Security

### 86. What is RBAC in Kubernetes?


RBAC controls which identities are allowed to perform which API actions on which Kubernetes resources.

The main objects are Role, ClusterRole, RoleBinding and ClusterRoleBinding.

---

| Role | ClusterRole |
|---|---|
| Namespace-scoped permission definition | Cluster-scoped/reusable permission definition |
| Used through RoleBinding | Can be bound through RoleBinding or ClusterRoleBinding |

**Example:** Example: Role `get/list` on Pods in `dev` is namespace-scoped; ClusterRole can define reusable or cluster-scoped permissions.

---

### 87. What is the difference between Role and ClusterRole?


A Role defines permissions within a namespace.

A ClusterRole is cluster-scoped and can define permissions for cluster-scoped resources or reusable permissions that can be bound within namespaces as well.

---

| RoleBinding | ClusterRoleBinding |
|---|---|
| Grants within a namespace | Grants referenced ClusterRole at cluster scope represented by the binding |

**Example:** A RoleBinding in `dev` can bind a ClusterRole while keeping the grant limited to `dev`.

---

### 88. What is the difference between RoleBinding and ClusterRoleBinding?


RoleBinding grants permissions within a namespace.

ClusterRoleBinding grants the referenced ClusterRole across the cluster scope represented by the binding.

---

**Example:** Example: bind a read-only ClusterRole to a ServiceAccount through a RoleBinding in one namespace.

---

### 89. Can a RoleBinding reference a ClusterRole?


Yes.

A RoleBinding can reference a ClusterRole and grant those permissions within the namespace of the RoleBinding.

This is useful for reusing a common permission definition while keeping the actual access namespace-scoped.

---

**Example:** Grant only the permissions required by the workload.

---

### 90. What is the principle of least privilege in Kubernetes?


Users, applications and service accounts should receive only the permissions required to perform their intended tasks.

For example, an application that only needs to read ConfigMaps should not receive cluster-admin privileges.

---

**Example:** A backend that only reads ConfigMaps should not receive `cluster-admin`.

---

### 91. What is a ServiceAccount?


A ServiceAccount provides an identity for workloads running inside the cluster.

Pods can use that identity when interacting with the Kubernetes API or other systems that rely on workload identity.

---

**Example:** A Pod can use a ServiceAccount identity when it needs Kubernetes API access.

---

### 92. How would you check why a ServiceAccount cannot access a Kubernetes resource?


I would inspect its Role/ClusterRole and bindings, then use:

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:<namespace>:<serviceaccount>
```

This directly tests whether the identity has the requested permission.

---

**Example:** Use `kubectl auth can-i` to test the effective permission of a ServiceAccount.

---

### 93. What is `kubectl auth can-i` used for?


It checks whether a particular user or ServiceAccount is authorized to perform an API operation.

For example:

```bash
kubectl auth can-i create deployments
```

or:

```bash
kubectl auth can-i get secrets \
  --as=system:serviceaccount:app:backend
```

---

**Example:** Example: `kubectl auth can-i get secrets --as=system:serviceaccount:app:backend`.

---

### 94. What is a NetworkPolicy?


NetworkPolicy defines rules controlling network traffic to and/or from Pods.

It can restrict traffic based on sources, destinations, namespaces, Pod selectors and ports, depending on the network plugin's capabilities.

---

| NetworkPolicy | Important point |
|---|---|
| Defines allowed Pod traffic | Requires network-plugin enforcement |
| Can control ingress/egress | Creating the object alone does not guarantee enforcement |

**Example:** NetworkPolicy enforcement depends on the network plugin.

---

### 95. Does Kubernetes NetworkPolicy work automatically in every cluster?


The Kubernetes API defines NetworkPolicy objects, but enforcement requires a network plugin that supports NetworkPolicy.

If the CNI does not implement the relevant policies, creating the object alone does not necessarily enforce traffic restrictions.

---

## 7. Control Plane and Kubernetes Internals

**Example:** Example: restrict database Pods so only selected application Pods can connect on the database port.

---

## 9. Kubernetes Troubleshooting

### 96. A Pod is stuck in `Pending`. How do you troubleshoot it?


I would start with:

```bash
kubectl get pod <pod> -o wide
kubectl describe pod <pod>
```

The Events section usually gives the scheduler reason.

Then I would check:

```bash
kubectl get nodes
kubectl describe nodes
```

and investigate resource requests, taints/tolerations, affinity, node selectors, topology constraints and PVC binding.

I would not randomly change configuration before identifying the scheduling reason.

---

**Example:** Start with `kubectl get pod <pod> -o wide` and `kubectl describe pod <pod>`; inspect Events for scheduling reasons.

---

### 97. A Pod is in `CrashLoopBackOff`. What is your troubleshooting approach?


I would first inspect current and previous container logs:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
```

Then:

```bash
kubectl describe pod <pod>
```

I would check exit codes, events, probes, environment variables, ConfigMaps/Secrets, mounted volumes, permissions, dependencies and resource limits.

The important point is that `CrashLoopBackOff` is a symptom. I need to identify why the process is exiting.

---

**Example:** Start with `kubectl logs <pod> --previous` and `kubectl describe pod <pod>`; identify why the process exits.

---

### 98. A Deployment is healthy, but users cannot access the application. What do you check?


I would trace the complete traffic path:

```text
Client
 ↓
DNS
 ↓
Load Balancer / Ingress
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
Application
```

Then I would check:

```bash
kubectl get ingress
kubectl get svc
kubectl get endpointslices
kubectl get pods -o wide
kubectl describe svc <service>
kubectl logs <pod>
```

I would verify DNS, ports, selectors, readiness, NetworkPolicies, Ingress rules, load balancer configuration and application listening ports.

---

**Example:** Trace the complete path instead of assuming a healthy Deployment means reachable application.

---

### 99. A Service exists, Pods are Running, but traffic is not reaching them. What could be wrong?


Running does not necessarily mean Ready.

I would first check whether the Service has endpoints:

```bash
kubectl get endpoints <service>
kubectl get endpointslices
```

If there are no endpoints, I would compare Service selectors with Pod labels and inspect readiness probes.

If endpoints exist, I would continue down the path and check Service ports, targetPort, NetworkPolicies, DNS, CNI/networking and application listeners.

---

**Example:** Running is not the same as Ready; inspect selectors, EndpointSlices, readiness and ports.

---

### 100. A Pod is Running but the application is still unavailable. Does `Running` mean the application is healthy?


No.

`Running` only tells me that the Pod has been assigned to a node and its containers have started according to the Pod lifecycle state.

The application can still be unhealthy, not Ready, unable to accept traffic, or returning application-level errors.

That is why I check readiness, liveness, logs, endpoints and the actual application behavior rather than relying only on `kubectl get pods`.

---

# Quick Interview Revision Matrix

| Topic | Key distinction |
|---|---|
| `nodeSelector` vs affinity | Simple exact matching vs expressive rules |
| Node affinity vs Pod affinity | Node labels vs other Pods |
| Affinity vs anti-affinity | Together vs apart |
| Taint vs toleration | Node restriction vs Pod permission |
| `NoSchedule` vs `NoExecute` | Block scheduling vs block + evict |
| Deployment vs ReplicaSet | Rollout manager vs replica manager |
| Deployment vs StatefulSet | Interchangeable replicas vs stable identity |
| Deployment vs DaemonSet | Replica count vs node coverage |
| Job vs CronJob | One finite execution vs scheduled executions |
| Pod vs container | Kubernetes unit vs runtime process |
| Pod vs Node | Workload unit vs worker machine |
| ClusterIP vs NodePort | Internal vs node-level exposure |
| NodePort vs LoadBalancer | Node port exposure vs external LB integration |
| `port` vs `targetPort` | Service port vs backend port |
| Service vs Pod IP | Stable virtual endpoint vs ephemeral Pod address |
| Endpoints vs EndpointSlice | Legacy endpoint object vs scalable endpoint representation |
| Readiness vs liveness | Traffic eligibility vs restart decision |
| Startup vs liveness | Startup protection vs ongoing health |
| Request vs limit | Scheduling baseline vs maximum |
| CPU vs memory limits | CPU can throttle; memory can cause OOM |
| ConfigMap vs Secret | Non-sensitive vs sensitive configuration |
| PV vs PVC | Storage resource vs storage request |
| PV vs `emptyDir` | Persistent storage vs Pod-lifetime temporary storage |
| Role vs ClusterRole | Namespace-scoped vs cluster-scoped/reusable permissions |
| RoleBinding vs ClusterRoleBinding | Namespace grant vs cluster-wide grant |
| Ingress vs Ingress Controller | Routing configuration vs implementation |
| API Server vs etcd | Kubernetes API/control gateway vs persistent state store |
| Scheduler vs kubelet | Selects node vs runs Pod on node |
| Desired vs actual state | Declared target vs current reality |

---

# Interview Answer Pattern

For scenario questions, avoid giving only a command.

Use this structure:

```text
1. State what the symptom means
2. Identify the likely categories of causes
3. Run commands to collect evidence
4. Interpret the output
5. Apply the fix
6. Verify the result
7. Explain prevention if relevant
```

For example:

> "First I would confirm the symptom with `kubectl get` and then use `kubectl describe` and logs to identify the actual failure. I would avoid making changes before confirming the root cause. Once I identify the issue, I would apply the minimal fix and verify the Pod, Service and application health."

This approach is much stronger in a DevOps interview than simply listing commands.

**Example:** A Pod can be Running while its application is failing readiness checks or returning application-level errors.

---

## Kubernetes Resource and Workload Quick Reference

| Object | Primary purpose |
|---|---|
| Pod | Smallest deployable compute unit |
| Deployment | Stateless rollout/replica management |
| StatefulSet | Stable identity/storage-aware workloads |
| DaemonSet | Node-level workload on eligible nodes |
| Job | Finite task |
| CronJob | Scheduled Job creation |
| Service | Stable network endpoint |
| Ingress | HTTP/HTTPS routing configuration |
| PVC | Storage request |
| ConfigMap | Non-sensitive configuration |
| Secret | Sensitive configuration/data |

## Kubernetes Networking Quick Reference

| Concept | Purpose |
|---|---|
| Pod IP | Address of an individual Pod |
| Service | Stable endpoint for a group of Pods |
| EndpointSlice | Current backend endpoint representation |
| ClusterIP | Internal Service access |
| NodePort | Node-level Service exposure |
| LoadBalancer | External LB integration |
| Ingress | HTTP/HTTPS routing |
| NetworkPolicy | Pod traffic controls |

## Kubernetes Health and Resource Quick Reference

| Concept | Interview meaning |
|---|---|
| Startup probe | Protect slow startup |
| Readiness probe | Controls traffic eligibility |
| Liveness probe | Detects unhealthy containers for restart action |
| Request | Scheduling/accounting baseline |
| Limit | Consumption ceiling |
| QoS | Pod resource-quality classification |
| OOMKilled | Container was killed due to memory pressure/limit behavior |

# Practical Examples

## 1. Deployment request flow

```text
kubectl apply
    ↓
API Server
    ↓
Authentication / Authorization / Admission
    ↓
Deployment Controller
    ↓
ReplicaSet
    ↓
Pod
    ↓
Scheduler
    ↓
Node / kubelet
    ↓
Container Runtime
    ↓
Container
```

## 2. Application traffic flow

```text
Client
  ↓
DNS
  ↓
Ingress / LoadBalancer
  ↓
Service
  ↓
EndpointSlice
  ↓
Ready Pod
  ↓
Application
```

## 3. Storage flow

```text
Pod → PVC → StorageClass / PV → Storage Backend
```

## 4. Scheduling flow

```text
Unscheduled Pod
      ↓
Scheduler evaluates nodes
      ↓
Filters constraints
      ↓
Selects and binds a node
      ↓
Kubelet runs the Pod
```

# Interview Answer Pattern

1. Explain what the symptom/state means.
2. Identify likely causes.
3. Collect evidence with commands.
4. Interpret the evidence.
5. Apply the minimal fix.
6. Verify the result.
7. Explain prevention where relevant.

```text
Symptom → Evidence → Root cause → Minimal fix → Verification → Prevention
```

## Accuracy Verification

The refactor was checked against current Kubernetes documentation covering cluster architecture/components, Pods, workloads, Services/networking, probes, resource management, NetworkPolicy and API access control. citeturn0search6turn0search11turn0search3turn0search10turn0search2turn0search7turn0search5turn0search8turn0search9
