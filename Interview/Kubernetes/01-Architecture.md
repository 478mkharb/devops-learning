# Kubernetes Architecture Interview Questions and Detailed Answers

## 1. What is Kubernetes?

**Answer:**

Kubernetes is an open-source container orchestration platform used to deploy, manage, scale, and maintain containerized applications.

It automates:

- Container deployment
- Scheduling containers on suitable nodes
- Scaling applications
- Service discovery
- Load balancing
- Self-healing
- Rolling updates and rollbacks
- Configuration and secret management

Kubernetes follows a **desired-state model**. The user defines the desired state, and Kubernetes continuously works to make the actual cluster state match that desired state.

---

## 2. What is Kubernetes architecture?

**Answer:**

Kubernetes architecture is based on a cluster model containing:

1. **Control Plane**
2. **Worker Nodes**

The Control Plane manages the cluster and makes decisions. Worker Nodes run application workloads inside Pods.

```text
                    Kubernetes Cluster
                           |
          -------------------------------------
          |                                   |
      Control Plane                       Worker Nodes
          |                         -------------------------
  -----------------------           | Node 1 | Node 2 | ... |
  | API Server           |          |        |        |     |
  | etcd                 |          | kubelet| kubelet|     |
  | Scheduler            |          | runtime| runtime|     |
  | Controller Manager   |          | kube-proxy            |
  | Cloud Controller     |          | Pods                  |
  -----------------------           -------------------------
```

---

## 3. What is a Kubernetes cluster?

**Answer:**

A Kubernetes cluster is a group of machines working together to run containerized applications.

A cluster normally contains:

- One or more Control Plane nodes
- One or more Worker Nodes
- A Kubernetes API endpoint
- A cluster data store, usually etcd
- Networking components
- A container runtime on worker nodes

The Control Plane manages the cluster, while Worker Nodes execute workloads.

---

## 4. What is the difference between the Control Plane and Worker Node?

**Answer:**

| Control Plane | Worker Node |
|---|---|
| Manages the cluster | Runs application workloads |
| Maintains desired state | Runs Pods |
| Makes scheduling decisions | Hosts containers |
| Stores cluster state through etcd | Reports node and Pod status |
| Runs API server, scheduler and controllers | Runs kubelet and container runtime |

### Control Plane components

- kube-apiserver
- etcd
- kube-scheduler
- kube-controller-manager
- cloud-controller-manager, when required

### Worker Node components

- kubelet
- Container runtime
- kube-proxy, when used
- Pods

---

## 5. What is the kube-apiserver?

**Answer:**

The kube-apiserver is the central entry point to the Kubernetes Control Plane.

All major Kubernetes operations go through the API server.

Examples:

```bash
kubectl get pods
kubectl create deployment nginx --image=nginx
kubectl apply -f deployment.yaml
```

The API server:

- Exposes the Kubernetes API
- Authenticates requests
- Authorizes requests
- Performs admission control
- Validates API objects
- Reads and writes cluster state in etcd
- Provides the communication interface between Kubernetes components

### Interview point

> The API server is the front door of the Kubernetes cluster. Users, kubectl, controllers, schedulers, and other components communicate with the cluster through the API server.

---

## 6. Is kube-apiserver stateless or stateful?

**Answer:**

The kube-apiserver process is generally treated as stateless because it does not permanently store the cluster state inside its own local filesystem.

The persistent cluster state is stored in etcd.

Multiple API server instances can therefore run behind a load balancer in a highly available cluster.

However, the API server still performs important processing such as authentication, authorization, validation, admission control, and API request handling.

---

## 7. What is etcd?

**Answer:**

etcd is a distributed, consistent key-value store used by Kubernetes to store cluster state.

It stores information such as:

- Pods
- Deployments
- ReplicaSets
- Services
- ConfigMaps
- Secrets
- Namespaces
- Nodes
- Roles and RoleBindings
- Custom Resources

### Example

When you create a Deployment, its desired configuration is stored in etcd through the API server.

### Important point

> etcd is the source of truth for Kubernetes cluster state.

The API server is the access layer, while etcd is the persistent storage layer.

---

## 8. Does kubectl communicate directly with etcd?

**Answer:**

No.

The normal communication path is:

```text
kubectl
   |
   v
kube-apiserver
   |
   v
etcd
```

Users and normal Kubernetes components should communicate with the API server rather than directly accessing etcd.

The API server handles authentication, authorization, validation, admission, and object management.

---

## 9. Why is etcd important?

**Answer:**

etcd is important because Kubernetes needs a reliable store for its desired and observed cluster state.

If etcd is lost without a backup:

- Kubernetes may lose cluster configuration
- Deployments and Services may disappear from the Control Plane view
- Recovery becomes difficult
- Running containers may continue temporarily, but cluster management is affected

Therefore, production clusters should have:

- etcd backups
- Encryption for sensitive data
- Monitoring
- Proper quorum planning
- Restricted access

---

## 10. What is etcd quorum?

**Answer:**

etcd uses a quorum-based consensus mechanism.

For a cluster of `N` etcd members, the quorum is:

```text
floor(N / 2) + 1
```

Examples:

| etcd members | Required quorum | Failure tolerance |
|---:|---:|---:|
| 1 | 1 | 0 |
| 3 | 2 | 1 |
| 5 | 3 | 2 |
| 7 | 4 | 3 |

A three-member etcd cluster can tolerate one member failure and still operate.

A five-member cluster can tolerate two member failures.

### Important point

> Odd numbers of etcd members are generally preferred because they provide better failure tolerance without unnecessary members.

---

## 11. What happens if etcd loses quorum?

**Answer:**

If etcd loses quorum, it cannot safely commit new updates.

As a result:

- New Kubernetes objects may not be created
- Updates may fail
- Controllers may not persist changes
- Scheduling and reconciliation may stop functioning correctly
- Existing workloads may continue running for some time

The cluster may appear partially operational, but Control Plane operations are impaired.

The solution is to restore enough etcd members to regain quorum or restore etcd from a valid backup.

---

## 12. What is kube-scheduler?

**Answer:**

The kube-scheduler assigns newly created, unscheduled Pods to suitable Worker Nodes.

The scheduler evaluates:

- CPU and memory requests
- Node availability
- Node selectors
- Node affinity
- Pod affinity and anti-affinity
- Taints and tolerations
- Topology constraints
- Volume requirements
- Scheduling policies

### Scheduler flow

```text
Pod created
   |
   v
API server stores Pod
   |
   v
Scheduler notices unscheduled Pod
   |
   v
Scheduler selects a suitable node
   |
   v
Binding decision is sent to API server
   |
   v
Kubelet on selected node creates the Pod
```

### Important point

> The scheduler selects a node. The kubelet actually creates and manages the Pod on that node.

---

## 13. Does the scheduler create containers?

**Answer:**

No.

The scheduler only selects the node where the Pod should run.

After scheduling:

1. The scheduler assigns the Pod to a node.
2. The kubelet on that node observes the assignment.
3. The kubelet asks the container runtime to create the Pod sandbox and containers.
4. The CNI configures networking.
5. The kubelet manages the Pod lifecycle.

---

## 14. What is kube-controller-manager?

**Answer:**

The kube-controller-manager runs multiple Kubernetes controllers in one process.

A controller continuously compares:

```text
Desired State vs Actual State
```

If the states differ, the controller takes corrective action.

Controllers managed by kube-controller-manager include:

- Node controller
- Replication controller
- EndpointSlice controller
- ServiceAccount controller
- Namespace controller
- Job controller
- Deployment-related control logic through ReplicaSets and other controllers

### Example

If a Deployment requires three replicas but only two Pods are running, the relevant controllers create another Pod.

---

## 15. What is a controller in Kubernetes?

**Answer:**

A controller is a control loop that watches Kubernetes resources and attempts to make the actual state match the desired state.

### Example

Desired state:

```text
replicas: 3
```

Actual state:

```text
running Pods: 2
```

The controller detects the difference and creates another Pod.

### Interview answer

> A Kubernetes controller is a reconciliation loop. It watches resources, compares desired and actual state, and takes action until the desired state is achieved.

---

## 16. What is reconciliation?

**Answer:**

Reconciliation is the continuous process of comparing the desired state with the actual state and correcting differences.

Example:

```text
Desired state: 3 nginx Pods
Actual state:  2 nginx Pods
Action:        Create 1 additional Pod
```

If a Pod later crashes:

```text
Desired state: 3 Pods
Actual state:  2 Pods
Action:        Create replacement Pod
```

This is the basis of Kubernetes self-healing.

---

## 17. What is cloud-controller-manager?

**Answer:**

The cloud-controller-manager integrates Kubernetes with a cloud provider.

It allows Kubernetes to use cloud-specific functionality without placing cloud-provider logic inside the core Kubernetes controllers.

It may manage:

- Cloud node information
- Cloud load balancers
- Cloud routes
- Node lifecycle integration
- Cloud provider-specific resources

Examples include integration with:

- AWS
- Azure
- Google Cloud
- Other supported cloud providers

### Important point

> cloud-controller-manager is used when Kubernetes needs cloud-provider-specific control logic.

---

## 18. What is kubelet?

**Answer:**

The kubelet is the main Kubernetes agent running on every Worker Node.

It is responsible for ensuring that the Pods assigned to its node are running and healthy.

The kubelet:

- Watches Pod specifications assigned to its node
- Communicates with the API server
- Requests the container runtime to create and manage containers
- Reports node and Pod status
- Runs liveness, readiness, and startup probe checks
- Mounts volumes
- Executes lifecycle hooks
- Restarts containers when required
- Invokes CNI plugins through the container runtime workflow

### Important point

> The kubelet is responsible for managing Pods on its node, but it does not normally schedule Pods.

---

## 19. Does kubelet create containers directly?

**Answer:**

No.

The kubelet communicates with the container runtime through the Container Runtime Interface, or CRI.

The general flow is:

```text
kubelet
   |
   v
CRI
   |
   v
container runtime
   |
   v
containers
```

Common container runtimes include:

- containerd
- CRI-O

The runtime creates the Pod sandbox and containers.

---

## 20. What is a container runtime?

**Answer:**

A container runtime is the software responsible for running containers.

It handles tasks such as:

- Pulling container images
- Creating containers
- Starting and stopping containers
- Managing container processes
- Managing container filesystems
- Reporting container status

Kubernetes communicates with the runtime through the CRI.

Examples:

- containerd
- CRI-O

---

## 21. What is the Container Runtime Interface?

**Answer:**

CRI is the interface between the kubelet and the container runtime.

It allows Kubernetes to work with different runtimes through a standard interface.

The kubelet uses CRI operations for tasks such as:

- Creating Pod sandboxes
- Starting containers
- Stopping containers
- Removing containers
- Fetching container status
- Retrieving logs
- Executing commands

---

## 22. What is kube-proxy?

**Answer:**

kube-proxy is a node-level component traditionally responsible for implementing Kubernetes Service traffic forwarding.

It watches Services and EndpointSlices and programs networking rules.

Depending on the mode, it may use:

- iptables
- IPVS
- Other supported mechanisms

Example:

```text
Client Pod
    |
    v
Service ClusterIP
    |
    v
kube-proxy rules
    |
    v
Backend Pod
```

### Important point

> kube-proxy is mainly associated with Service networking, not basic direct Pod-to-Pod communication.

Some CNI implementations, such as Cilium, can replace kube-proxy using eBPF.

---

## 23. What are the main components of a Worker Node?

**Answer:**

A Worker Node commonly contains:

1. **kubelet**
2. **Container runtime**
3. **kube-proxy**, when used
4. **CNI networking components**
5. **Pods**

### Responsibilities

| Component | Responsibility |
|---|---|
| kubelet | Manages Pods assigned to the node |
| Container runtime | Runs containers |
| kube-proxy | Traditionally implements Service forwarding |
| CNI | Configures Pod networking |
| Pods | Run application containers |

---

## 24. What is a Pod?

**Answer:**

A Pod is the smallest deployable unit in Kubernetes.

A Pod contains one or more containers that share:

- Network namespace
- Pod IP address
- Local storage volumes
- Lifecycle and scheduling context

Containers inside the same Pod communicate using:

```text
localhost
```

Example:

```text
Application container: localhost:8080
Sidecar container:     localhost:9090
```

A Pod is normally scheduled as one unit onto a single node.

---

## 25. Why does Kubernetes use Pods instead of directly managing containers?

**Answer:**

Kubernetes uses Pods because some containers need to work closely together and share resources.

Containers in the same Pod can share:

- Network namespace
- IP address
- Volumes
- Lifecycle
- Scheduling placement

Common examples:

- Application container + logging sidecar
- Application container + proxy sidecar
- Application container + monitoring agent

The Pod provides the execution and management boundary for these tightly coupled containers.

---

## 26. What is the Pod sandbox?

**Answer:**

The Pod sandbox is the environment created for a Pod before its application containers run.

It provides the Pod-level isolation and networking context.

The container runtime creates the sandbox, and the networking plugin configures the Pod network.

The general flow is:

```text
kubelet
   |
   v
container runtime creates Pod sandbox
   |
   v
CNI configures Pod network
   |
   v
application containers are created
```

---

## 27. What happens when a Pod is created?

**Answer:**

The process is generally:

1. A user submits a manifest using `kubectl`.
2. The request reaches the kube-apiserver.
3. The API server authenticates and authorizes the request.
4. Admission controllers and validation are applied.
5. The object is stored in etcd.
6. The scheduler notices an unscheduled Pod.
7. The scheduler selects a suitable node.
8. The binding is recorded through the API server.
9. The kubelet on that node notices the assigned Pod.
10. The kubelet asks the container runtime to create the Pod sandbox.
11. The runtime invokes the CNI plugin.
12. CNI configures the Pod interface, IP address, routes, and connectivity.
13. The runtime pulls images if required.
14. The runtime starts the containers.
15. The kubelet performs health checks.
16. Pod status is reported to the API server.

---

## 28. What happens when you run `kubectl apply -f deployment.yaml`?

**Answer:**

The flow is:

```text
kubectl
   |
   v
kube-apiserver
   |
   v
Authentication and Authorization
   |
   v
Admission and Validation
   |
   v
etcd
   |
   v
Deployment Controller
   |
   v
ReplicaSet
   |
   v
Pod
   |
   v
Scheduler
   |
   v
Selected Worker Node
   |
   v
kubelet
   |
   v
Container Runtime
   |
   v
CNI + Container
```

The Deployment controller creates or updates a ReplicaSet. The ReplicaSet ensures that the required number of Pods exists. The scheduler assigns each unscheduled Pod to a node, and the kubelet starts it.

---

## 29. What is the difference between Deployment, ReplicaSet and Pod?

**Answer:**

| Resource | Main responsibility |
|---|---|
| Pod | Runs one or more containers |
| ReplicaSet | Maintains the desired number of identical Pods |
| Deployment | Manages ReplicaSets and application rollouts |

### Relationship

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
    |
    v
Containers
```

A Deployment is normally used for stateless applications because it provides:

- Scaling
- Rolling updates
- Rollbacks
- Replica management

---

## 30. What is the desired state in Kubernetes?

**Answer:**

Desired state is the condition that the user wants Kubernetes to maintain.

Example:

```yaml
replicas: 3
image: nginx:1.27
```

This means:

- Three replicas should exist
- They should use the specified image

Kubernetes continuously works to maintain this state even if:

- A container crashes
- A Pod is deleted
- A node becomes unavailable
- A new version is deployed

---

## 31. What is the actual state?

**Answer:**

Actual state is the current condition of the cluster.

Example:

```text
Desired state: 3 replicas
Actual state:  2 running Pods
```

Kubernetes controllers compare the two states and take corrective action.

The actual state is observed from the cluster through the API server and status information reported by components such as kubelet.

---

## 32. What is self-healing in Kubernetes?

**Answer:**

Self-healing means Kubernetes automatically attempts to recover from failures.

Examples:

- Restarting failed containers
- Recreating deleted Pods
- Replacing unhealthy Pods
- Rescheduling workloads after node failure, when possible
- Maintaining the desired replica count

For example, if a Deployment requires three replicas and one Pod is deleted, the ReplicaSet creates a replacement Pod.

---

## 33. What is the difference between a Kubernetes object and a Kubernetes component?

**Answer:**

A **Kubernetes object** represents the desired or observed state of a resource.

Examples:

- Pod
- Deployment
- Service
- ConfigMap
- Secret
- Namespace
- Job

A **Kubernetes component** is software that operates the cluster.

Examples:

- kube-apiserver
- kube-scheduler
- kubelet
- kube-controller-manager
- etcd
- container runtime

### Simple distinction

> Objects are resources managed by Kubernetes. Components are the software that manages those resources.

---

## 34. What are Kubernetes API objects?

**Answer:**

API objects are persistent entities in the Kubernetes API.

They describe resources and their desired or observed state.

Most objects contain:

- `apiVersion`
- `kind`
- `metadata`
- `spec`
- `status`, when applicable

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
```

The `spec` normally describes the desired state, while `status` describes the current observed state.

---

## 35. What is the difference between `spec` and `status`?

**Answer:**

| Field | Meaning |
|---|---|
| `spec` | Desired state |
| `status` | Observed current state |

Example:

```yaml
spec:
  replicas: 3
```

This means the user wants three replicas.

```yaml
status:
  availableReplicas: 2
```

This means only two replicas are currently available.

Controllers use this information to reconcile the cluster.

---

## 36. What is the role of metadata in a Kubernetes object?

**Answer:**

Metadata identifies and organizes an object.

Common metadata fields include:

- `name`
- `namespace`
- `labels`
- `annotations`
- `uid`
- `resourceVersion`
- `creationTimestamp`

### Example

```yaml
metadata:
  name: frontend
  namespace: production
  labels:
    app: frontend
```

Labels are used for selection and grouping. Annotations store additional non-identifying information.

---

## 37. What is a Namespace?

**Answer:**

A Namespace provides a logical separation of resources within a Kubernetes cluster.

Namespaces are commonly used to separate:

- Development
- Testing
- Staging
- Production
- Different teams
- Different applications

Namespaces can be used with:

- ResourceQuota
- LimitRange
- RBAC
- NetworkPolicy

### Important point

> A Namespace is a logical boundary, not a separate Kubernetes cluster or automatically a complete security boundary.

---

## 38. What are labels and selectors?

**Answer:**

Labels are key-value pairs attached to Kubernetes objects.

Example:

```yaml
labels:
  app: frontend
  environment: production
```

Selectors are used to find objects with matching labels.

A Service may select:

```yaml
selector:
  app: frontend
```

The Service then routes traffic to matching Pods.

Labels are also used by:

- Deployments
- ReplicaSets
- Services
- NetworkPolicies
- Scheduling rules
- Monitoring systems

---

## 39. What are annotations?

**Answer:**

Annotations are key-value metadata used to store additional information that is not normally used for selecting objects.

Examples:

- Ingress controller configuration
- Tool-specific metadata
- Deployment information
- Documentation links
- Configuration hints

### Difference

| Labels | Annotations |
|---|---|
| Used for selection and grouping | Store additional metadata |
| Should be suitable for querying | Not normally used for selection |
| Often used by Services and controllers | Often used by tools and controllers |

---

## 40. What is a ServiceAccount?

**Answer:**

A ServiceAccount provides an identity for processes running inside Pods.

Applications use ServiceAccounts when they need to communicate with the Kubernetes API or access resources through configured permissions.

A ServiceAccount can be associated with:

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

The permissions are controlled through RBAC.

### Important point

> A ServiceAccount identifies an application or Pod workload. It is different from a human user account.

---

## 41. What is RBAC?

**Answer:**

RBAC stands for Role-Based Access Control.

It controls who can perform which actions on which Kubernetes resources.

RBAC uses:

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding
- User, Group or ServiceAccount subjects

Example permissions:

```text
get Pods
list Pods
create Deployments
delete Services
```

A Role is namespace-scoped. A ClusterRole can define cluster-scoped permissions.

---

## 42. What are admission controllers?

**Answer:**

Admission controllers are components that intercept API requests after authentication and authorization but before the object is persisted.

They can:

- Validate requests
- Mutate objects
- Enforce policies
- Apply defaults
- Reject unsafe configurations

Examples include controls related to:

- Resource limits
- Pod security
- Image policies
- Namespace restrictions
- ServiceAccount behavior

### Request flow

```text
Request
   |
   v
Authentication
   |
   v
Authorization
   |
   v
Admission Control
   |
   v
Validation
   |
   v
etcd
```

---

## 43. What is the difference between authentication, authorization and admission?

**Answer:**

| Stage | Main question |
|---|---|
| Authentication | Who are you? |
| Authorization | Are you allowed to perform this action? |
| Admission | Should this request be accepted, modified or rejected? |

### Example

1. Authentication identifies the user.
2. Authorization checks whether the user can create a Deployment.
3. Admission checks policy and may modify or reject the Deployment.
4. If accepted, the object is stored.

---

## 44. What are static Pods?

**Answer:**

Static Pods are managed directly by the kubelet on a specific node rather than by a normal controller such as a Deployment.

The kubelet watches a configured static Pod manifest directory.

Static Pods are commonly used by cluster bootstrapping tools for Control Plane components such as:

- kube-apiserver
- kube-scheduler
- kube-controller-manager
- etcd

### Important point

> Static Pods are bound to a node and are created by the kubelet from local manifest files.

---

## 45. What is the difference between a static Pod and a Deployment-managed Pod?

**Answer:**

| Static Pod | Deployment-managed Pod |
|---|---|
| Managed by kubelet | Managed through API controllers |
| Defined in local manifest files | Defined in API objects |
| Bound to one node | Can be scheduled across nodes |
| Common for Control Plane bootstrapping | Common for applications |
| Not managed by a ReplicaSet | Usually managed by a ReplicaSet |

---

## 46. What is leader election in Kubernetes?

**Answer:**

Leader election allows multiple instances of a component to run for high availability while only one active instance performs a particular control function at a time.

It is commonly used by controllers and other highly available components.

The leader periodically renews its lease. If it fails, another instance can acquire leadership.

This prevents multiple instances from performing conflicting active work.

---

## 47. How does Kubernetes maintain high availability?

**Answer:**

A highly available Kubernetes cluster generally uses:

- Multiple Control Plane nodes
- Multiple API server instances
- A load balancer in front of API servers
- Multiple etcd members
- Redundant controller and scheduler instances
- Multiple Worker Nodes
- Replicated application Pods
- Backups and monitoring

### Example

```text
                 Load Balancer
                      |
          -------------------------
          |          |            |
      API Server  API Server  API Server
          |          |            |
          -------- etcd cluster ---
```

The exact design depends on the environment and managed Kubernetes service.

---

## 48. What happens when a Worker Node fails?

**Answer:**

When a Worker Node fails:

1. The kubelet stops reporting healthy status.
2. The Node controller detects the problem.
3. The node may become `NotReady`.
4. Pods on the failed node may become unavailable.
5. Controllers attempt to maintain the desired replica count.
6. Replacement Pods may be scheduled on healthy nodes.
7. Services route traffic to available endpoints.

The exact timing depends on node monitoring, eviction settings, workload type, storage, and cluster configuration.

---

## 49. What happens when a Pod crashes?

**Answer:**

The result depends on the Pod’s owner and restart policy.

### If a container crashes inside a Pod

The kubelet may restart the container according to the Pod restart policy.

### If a Pod is deleted

A controller such as a ReplicaSet or StatefulSet may create a replacement.

### If the Pod belongs to a Job

The Job controller may create another attempt depending on its configuration.

### Important point

> Kubernetes does not normally repair the same Pod object forever. Controllers often replace failed or deleted Pods with new Pod objects.

---

## 50. What is the difference between restarting a container and recreating a Pod?

**Answer:**

### Container restart

The same Pod remains, but a container inside it is restarted.

### Pod recreation

The old Pod is removed or becomes unavailable, and a controller creates a new Pod.

A new Pod usually receives:

- A new Pod UID
- Potentially a new Pod IP
- A new placement decision, if rescheduled

This distinction is important when troubleshooting application state and networking.

---

## 51. What is the Kubernetes control loop?

**Answer:**

The Kubernetes control loop is the repeated process through which components observe the cluster and make changes.

```text
Observe actual state
        |
        v
Compare with desired state
        |
        v
Take corrective action
        |
        v
Observe again
```

This loop is implemented by controllers, kubelet, scheduler, and other components.

---

## 52. What is the difference between declarative and imperative management?

**Answer:**

### Declarative management

You describe the desired state in a manifest.

```bash
kubectl apply -f deployment.yaml
```

Kubernetes determines the required actions.

### Imperative management

You directly issue commands describing the action.

```bash
kubectl create deployment nginx --image=nginx
```

Declarative management is generally preferred for production because manifests can be version-controlled and reviewed.

---

## 53. What is the difference between `kubectl apply` and `kubectl create`?

**Answer:**

`kubectl create` is generally used to create a resource.

```bash
kubectl create deployment nginx --image=nginx
```

If the resource already exists, the command usually fails.

`kubectl apply` is used to create or update a resource based on a declared configuration.

```bash
kubectl apply -f deployment.yaml
```

It is commonly used in GitOps and CI/CD workflows.

---

## 54. What is the Kubernetes API server request flow?

**Answer:**

A typical API request follows this sequence:

```text
Client
  |
  v
API Server
  |
  v
Authentication
  |
  v
Authorization
  |
  v
Admission Controllers
  |
  v
Validation and Defaulting
  |
  v
etcd
  |
  v
API Response
```

After the object is stored, watches notify relevant controllers and components.

---

## 55. How do Kubernetes components communicate with each other?

**Answer:**

Kubernetes components communicate primarily through the Kubernetes API server.

Examples:

- kubectl → API server
- Scheduler → API server
- Controllers → API server
- kubelet → API server
- API server → etcd

Some node-level communication also occurs between:

- kubelet and container runtime through CRI
- kubelet/runtime and CNI plugins
- kube-proxy or eBPF datapath and the Linux networking stack

### Important point

> The API server is the central coordination point for Kubernetes control-plane communication.

---

## 56. What is a watch in Kubernetes?

**Answer:**

A watch allows a client to receive notifications when Kubernetes resources change.

Instead of repeatedly querying the API server, a controller can watch for events such as:

- Added
- Modified
- Deleted

Controllers use watches to react quickly to changes.

Example:

> The Deployment controller watches Deployments and ReplicaSets so it can reconcile changes.

---

## 57. What is the difference between polling and watching?

**Answer:**

### Polling

A client repeatedly asks for the current state.

```text
Request → Response
Request → Response
Request → Response
```

### Watching

A client establishes a watch and receives change events.

```text
Watch connection
    |
    +-- Added
    +-- Modified
    +-- Deleted
```

Watching is more efficient for controllers because they do not need to continuously request the complete state.

---

## 58. What is a Kubernetes finalizer?

**Answer:**

A finalizer is metadata that prevents an object from being fully deleted until a required cleanup operation is completed.

Examples of cleanup may include:

- Removing external cloud resources
- Cleaning up dependent resources
- Releasing infrastructure
- Performing controller-specific actions

If a finalizer is not removed because cleanup fails, the object may remain in a terminating state.

---

## 59. What is an OwnerReference?

**Answer:**

An OwnerReference identifies the object that owns another Kubernetes object.

Example:

```text
Deployment
   |
   v
ReplicaSet
   |
   v
Pod
```

A Pod may have an OwnerReference pointing to its ReplicaSet.

OwnerReferences allow Kubernetes to understand relationships and support garbage collection.

---

## 60. What is garbage collection in Kubernetes?

**Answer:**

Garbage collection is the process of removing dependent objects when their owners are deleted, according to Kubernetes ownership rules.

For example:

```text
Deployment deleted
        |
        v
ReplicaSet may be deleted
        |
        v
Owned Pods may be deleted
```

The behavior depends on deletion propagation settings and ownership relationships.

---

## 61. What are resource requests and limits?

**Answer:**

### Resource request

The amount of CPU or memory Kubernetes uses when scheduling a Pod.

### Resource limit

The maximum resource amount that a container is allowed to use, subject to resource type and runtime behavior.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

The scheduler uses requests to determine whether a node has enough available capacity.

---

## 62. What are Kubernetes QoS classes?

**Answer:**

Kubernetes assigns a Quality of Service class to Pods based on their resource requests and limits.

The main classes are:

1. Guaranteed
2. Burstable
3. BestEffort

### Guaranteed

All containers have CPU and memory requests and limits, and they match for each resource.

### Burstable

The Pod has some resource requests or limits but does not meet Guaranteed requirements.

### BestEffort

No CPU or memory requests or limits are specified.

QoS classes influence eviction behavior during resource pressure.

---

## 63. What are liveness, readiness and startup probes?

**Answer:**

### Liveness probe

Checks whether the container should be restarted.

### Readiness probe

Checks whether the Pod should receive traffic from Services.

### Startup probe

Gives slow-starting applications time to initialize before liveness and readiness checks take effect.

| Probe | Main purpose |
|---|---|
| Liveness | Restart unhealthy container |
| Readiness | Remove Pod from traffic endpoints |
| Startup | Detect successful application startup |

---

## 64. What is the difference between `Running` and `Ready`?

**Answer:**

`Running` means the Pod has been assigned to a node and its containers have started or are running according to Pod status.

`Ready` means the Pod has passed its readiness conditions and is considered eligible to receive traffic.

A Pod can be:

```text
Phase: Running
Ready: False
```

For example, an application may be running but still waiting for a database connection.

---

## 65. What is a Namespace versus a Node?

**Answer:**

| Namespace | Node |
|---|---|
| Logical grouping of resources | Physical or virtual machine |
| Used for organization and policy | Runs Pods |
| Does not run workloads itself | Provides compute resources |
| Cluster-scoped logical concept | Infrastructure component |

A Namespace can contain resources scheduled across many Nodes.

---

## 66. What is a managed Kubernetes service?

**Answer:**

A managed Kubernetes service is a cloud provider offering where the provider manages some or all Control Plane responsibilities.

Examples include:

- Amazon EKS
- Azure AKS
- Google Kubernetes Engine

Depending on the service, the provider may manage:

- API servers
- etcd
- Control Plane availability
- Control Plane upgrades
- Control Plane security

The customer may still manage:

- Worker Nodes
- Node groups
- Applications
- RBAC
- Networking configuration
- Storage
- Monitoring
- Security policies

---

## 67. What is the difference between self-managed and managed Kubernetes?

**Answer:**

### Self-managed Kubernetes

The organization manages:

- Control Plane
- etcd
- Worker Nodes
- Upgrades
- Backups
- Security
- Monitoring

### Managed Kubernetes

The cloud provider manages much of the Control Plane.

The customer generally focuses more on:

- Worker infrastructure
- Applications
- Networking
- Storage
- Security
- Observability
- Deployment automation

---

## 68. How would you troubleshoot a Kubernetes Control Plane issue?

**Answer:**

Start by checking:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get componentstatuses
kubectl get events -A --sort-by=.lastTimestamp
```

Then inspect:

```bash
kubectl cluster-info
kubectl get --raw='/readyz?verbose'
kubectl get --raw='/livez?verbose'
```

On self-managed clusters, inspect:

- kube-apiserver logs
- kube-scheduler logs
- kube-controller-manager logs
- etcd health
- Certificate expiration
- Disk space
- CPU and memory
- Network connectivity
- Load balancer health

For static Pods:

```bash
kubectl -n kube-system get pods
crictl ps
crictl logs <container-id>
```

---

## 69. How would you troubleshoot a Worker Node that is `NotReady`?

**Answer:**

Check the node:

```bash
kubectl get nodes
kubectl describe node <node-name>
```

Check events:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

On the node, inspect:

```bash
systemctl status kubelet
journalctl -u kubelet -n 100 --no-pager
systemctl status containerd
df -h
free -m
ip addr
ip route
```

Common causes include:

- kubelet stopped
- Container runtime failure
- Disk pressure
- Memory pressure
- Network problems
- Certificate problems
- CNI failure
- Time synchronization issues
- Node resource exhaustion

---

## 70. Explain Kubernetes architecture in one interview answer.

**Answer:**

> Kubernetes uses a Control Plane and Worker Node architecture. The Control Plane manages the cluster through components such as kube-apiserver, etcd, kube-scheduler, and kube-controller-manager. The API server is the central entry point, while etcd stores the cluster’s persistent state. The scheduler assigns unscheduled Pods to suitable nodes, and controllers continuously reconcile the desired state with the actual state.
>
> Worker Nodes run kubelet, a container runtime, networking components, and application Pods. The kubelet manages Pods assigned to the node and communicates with the container runtime through CRI. The runtime creates the Pod sandbox and containers, while the CNI configures Pod networking. kube-proxy traditionally implements Service forwarding, although some CNIs use eBPF instead.
>
> When a user applies a Deployment, the request goes through the API server and is stored in etcd. The Deployment controller creates a ReplicaSet, the ReplicaSet creates Pods, the scheduler selects nodes, and the kubelet starts the containers. Kubernetes then continuously monitors the cluster and takes corrective action to maintain the desired state.
