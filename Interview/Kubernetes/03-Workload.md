# Kubernetes Workloads Interview Questions and Detailed Answers

## 1. What is a Kubernetes workload?

**Answer:**

A workload is an application or task running on Kubernetes.

Kubernetes provides workload resources to manage different application patterns:

- Pod
- ReplicaSet
- Deployment
- StatefulSet
- DaemonSet
- Job
- CronJob

Each workload type is designed for a different operational requirement.

---

## 2. What is a Pod?

**Answer:**

A Pod is the smallest deployable unit in Kubernetes.

A Pod contains one or more containers that share:

- The same network namespace
- The same Pod IP address
- Volumes attached to the Pod
- Scheduling and lifecycle context

Containers inside the same Pod communicate using `localhost`.

A Pod is normally scheduled onto one node and its containers run together.

---

## 3. Why does Kubernetes use Pods instead of directly managing containers?

**Answer:**

Kubernetes uses Pods because some containers need to operate together as one application unit.

Containers in the same Pod can share:

- Network namespace
- IP address
- Storage volumes
- Lifecycle
- Scheduling placement

A common example is:

```text
Pod
├── Application container
└── Logging or proxy sidecar container
```

The Pod provides the management and scheduling boundary.

---

## 4. Can a Pod contain multiple containers?

**Answer:**

Yes.

A Pod can contain multiple containers, but they should normally be closely related and need to share resources.

Examples:

- Application container and logging sidecar
- Application container and service-mesh proxy
- Application container and configuration reloader

Multiple containers in a Pod:

- Share the same IP
- Share the same network namespace
- Can communicate through `localhost`
- Can share mounted volumes

Unrelated applications should generally be placed in separate Pods.

---

## 5. What is the difference between a Pod and a container?

**Answer:**

| Pod | Container |
|---|---|
| Kubernetes deployment unit | Runtime execution unit |
| Can contain one or more containers | Runs one application process or process group |
| Has a Pod IP | Usually shares the Pod network |
| Managed by Kubernetes | Managed by the container runtime |
| Provides shared lifecycle and resources | Executes application code |

A Pod is not itself a container. It is a Kubernetes abstraction that groups containers.

---

## 6. What is the lifecycle of a Pod?

**Answer:**

The main Pod phases are:

- Pending
- Running
- Succeeded
- Failed
- Unknown

### Pending

The Pod has been accepted but is not yet running. It may be waiting for:

- Scheduling
- Image pulling
- Volume attachment
- Resource availability

### Running

The Pod has been assigned to a node and at least one container is running or starting.

### Succeeded

All containers completed successfully and will not be restarted.

### Failed

All containers have terminated and at least one terminated unsuccessfully.

### Unknown

The system cannot determine the Pod state, often because communication with the node is unavailable.

---

## 7. What is the difference between Pod phase and container state?

**Answer:**

Pod phase describes the overall high-level lifecycle of the Pod.

Container state describes the condition of an individual container.

Container states include:

- Waiting
- Running
- Terminated

Example:

```text
Pod phase: Running
Container state: Waiting
Reason: CrashLoopBackOff
```

A Pod can have a `Running` phase while one container is waiting, restarting, or failing.

---

## 8. What is a ReplicaSet?

**Answer:**

A ReplicaSet maintains a stable number of identical Pod replicas.

Example:

```yaml
spec:
  replicas: 3
```

The ReplicaSet continuously checks the number of matching Pods.

If the desired count is three but only two exist, it creates another Pod.

If four matching Pods exist, it removes or allows the excess Pods to be reduced according to controller behavior.

---

## 9. What is a Deployment?

**Answer:**

A Deployment manages a set of Pods through ReplicaSets.

It provides:

- Declarative application updates
- Scaling
- Rolling updates
- Rollbacks
- Replica management
- Revision history

The normal relationship is:

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

Deployments are commonly used for stateless applications.

---

## 10. Why is a Deployment preferred over creating Pods directly?

**Answer:**

A manually created Pod is not automatically recreated if it is deleted.

A Deployment provides:

- Self-healing through ReplicaSets
- Scaling
- Rolling updates
- Rollbacks
- Desired replica management
- Version tracking

Therefore, production applications are usually managed through Deployments rather than standalone Pods.

---

## 11. What happens if a Pod created by a Deployment is deleted?

**Answer:**

The ReplicaSet controlled by the Deployment detects that the number of Pods is below the desired count.

It creates a replacement Pod.

The replacement Pod may have:

- A different Pod name
- A different Pod UID
- A different Pod IP
- A different node placement

The Deployment maintains the desired number of replicas, not the identity of a particular Pod.

---

## 12. What happens if a container inside a Pod crashes?

**Answer:**

The kubelet and container runtime handle the container restart according to the Pod’s restart policy.

For a Deployment-managed Pod, the usual restart policy is `Always`.

The container may be restarted inside the same Pod.

This is different from Pod recreation.

### Container restart

```text
Same Pod
└── Container restarted
```

### Pod recreation

```text
Old Pod removed
New Pod created
```

A recreated Pod normally receives a new UID and potentially a new IP.

---

## 13. What is the difference between a Deployment and a ReplicaSet?

**Answer:**

| Deployment | ReplicaSet |
|---|---|
| Manages application releases | Maintains Pod replica count |
| Creates and manages ReplicaSets | Creates and manages Pods |
| Supports rolling updates and rollback | Does not provide full rollout management |
| Recommended for stateless applications | Usually managed by a Deployment |

A Deployment uses ReplicaSets internally.

---

## 14. What is a rolling update?

**Answer:**

A rolling update replaces old Pods with new Pods gradually instead of stopping the entire application at once.

Example:

```text
Old version: 3 Pods
New version: 3 Pods
```

Kubernetes gradually:

1. Creates new-version Pods
2. Waits for them to become available
3. Removes old-version Pods
4. Continues until all old Pods are replaced

Rolling updates help reduce downtime.

---

## 15. What is the Recreate deployment strategy?

**Answer:**

The Recreate strategy terminates existing Pods before creating new Pods.

Example:

```text
Stop old Pods
     |
     v
Create new Pods
```

This can cause downtime, but it may be useful when old and new application versions cannot run simultaneously.

---

## 16. Rolling update vs Recreate strategy

**Answer:**

| RollingUpdate | Recreate |
|---|---|
| Replaces Pods gradually | Deletes old Pods first |
| Usually supports low downtime | Can cause downtime |
| Old and new versions may coexist | Versions do not normally coexist |
| Default Deployment strategy | Must be explicitly selected |

---

## 17. What are `maxSurge` and `maxUnavailable`?

**Answer:**

These settings control the behavior of a Deployment rolling update.

### `maxSurge`

Defines how many Pods above the desired replica count can temporarily exist.

### `maxUnavailable`

Defines how many Pods can be unavailable during the update.

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

This allows one extra Pod while ensuring that no desired Pod is unavailable, subject to readiness and cluster capacity.

Values can be specified as numbers or percentages.

---

## 18. How do you scale a Deployment?

**Answer:**

Using the command line:

```bash
kubectl scale deployment frontend --replicas=5
```

Or declaratively:

```yaml
spec:
  replicas: 5
```

Then apply the manifest:

```bash
kubectl apply -f deployment.yaml
```

The Deployment updates its ReplicaSet, which creates or removes Pods to reach five replicas.

---

## 19. How do you check the rollout status of a Deployment?

**Answer:**

Use:

```bash
kubectl rollout status deployment/frontend
```

Other useful commands:

```bash
kubectl get deployment frontend
kubectl get replicasets
kubectl get pods
kubectl rollout history deployment/frontend
```

To inspect the Deployment:

```bash
kubectl describe deployment frontend
```

---

## 20. How do you rollback a Deployment?

**Answer:**

First check rollout history:

```bash
kubectl rollout history deployment/frontend
```

Rollback to the previous revision:

```bash
kubectl rollout undo deployment/frontend
```

Rollback to a specific revision:

```bash
kubectl rollout undo deployment/frontend --to-revision=2
```

Check the rollback:

```bash
kubectl rollout status deployment/frontend
```

---

## 21. What is a StatefulSet?

**Answer:**

A StatefulSet manages stateful applications that require stable identity or persistent storage.

It provides:

- Stable Pod names
- Stable network identity
- Ordered creation and termination
- Ordered rolling updates, according to configuration
- Persistent volume association

Example Pod names:

```text
database-0
database-1
database-2
```

StatefulSets are commonly used for:

- Databases
- Distributed databases
- Message brokers
- Clustered systems

---

## 22. Deployment vs StatefulSet

**Answer:**

| Deployment | StatefulSet |
|---|---|
| Usually for stateless applications | For stateful applications |
| Pods are interchangeable | Pods have stable identities |
| Pod names are generated | Predictable ordinal names |
| Storage is not inherently tied to Pod identity | Can associate storage with Pod identity |
| Ordering is not generally important | Supports ordered operations |

A StatefulSet does not automatically make an application stateful. The application must still correctly manage data and replication.

---

## 23. What is a StatefulSet Pod identity?

**Answer:**

StatefulSet Pods receive stable ordinal identities.

For a StatefulSet named `database`:

```text
database-0
database-1
database-2
```

If `database-1` is recreated, it normally retains the same name and ordinal identity.

This is useful for clustered applications that need stable member identities.

---

## 24. What is a headless Service and why is it used with StatefulSets?

**Answer:**

A headless Service is created with:

```yaml
clusterIP: None
```

It does not provide a virtual ClusterIP.

Instead, DNS returns the individual Pod addresses.

StatefulSets commonly use headless Services so that each Pod can be discovered through stable DNS names.

Example:

```text
database-0.database.default.svc.cluster.local
database-1.database.default.svc.cluster.local
```

---

## 25. What is a DaemonSet?

**Answer:**

A DaemonSet ensures that a Pod runs on each eligible node.

Typical use cases include:

- Node monitoring agents
- Log collection agents
- Network plugins
- Storage plugins
- Security agents

Example:

```text
Node 1 → DaemonSet Pod
Node 2 → DaemonSet Pod
Node 3 → DaemonSet Pod
```

When a new eligible node joins the cluster, the DaemonSet normally creates its Pod there.

---

## 26. Does a DaemonSet run exactly one Pod on every node?

**Answer:**

Not necessarily every node.

A DaemonSet runs one Pod on each node that matches its scheduling rules.

Nodes may be excluded because of:

- Node selectors
- Node affinity
- Taints and tolerations
- Other scheduling constraints

A DaemonSet may also be configured to run more than one Pod per node through advanced patterns, but the normal behavior is one Pod per eligible node.

---

## 27. Deployment vs DaemonSet

**Answer:**

| Deployment | DaemonSet |
|---|---|
| Runs a desired number of replicas | Runs a Pod on each eligible node |
| Used for application workloads | Used for node-level services |
| Scheduler distributes replicas | DaemonSet ensures node coverage |
| Scaling is replica-based | Scaling follows eligible node count |

---

## 28. What is a Job?

**Answer:**

A Job creates one or more Pods and ensures that a specified task completes successfully.

Jobs are used for finite tasks such as:

- Database migrations
- Batch processing
- Data exports
- One-time scripts
- Report generation

A Job is complete when the required successful Pod completions are achieved.

---

## 29. What is a CronJob?

**Answer:**

A CronJob creates Jobs according to a schedule.

Example:

```yaml
schedule: "0 2 * * *"
```

This runs a Job approximately every day at 2:00 AM according to the controller’s time configuration.

Common use cases:

- Backups
- Cleanup tasks
- Scheduled reports
- Periodic data processing

A CronJob creates Jobs; it does not directly run application containers itself.

---

## 30. Job vs CronJob

**Answer:**

| Job | CronJob |
|---|---|
| Runs a task to completion | Creates Jobs on a schedule |
| Usually one-time or manually triggered | Recurring execution |
| Tracks successful completions | Tracks scheduled Job creation |
| Used for batch tasks | Used for scheduled batch tasks |

---

## 31. What is `restartPolicy`?

**Answer:**

`restartPolicy` controls how containers in a Pod are restarted.

Allowed values are:

- `Always`
- `OnFailure`
- `Never`

### Always

Containers are restarted whenever they terminate, subject to kubelet behavior.

### OnFailure

Containers are restarted only when they terminate with a failure.

### Never

Containers are not restarted after termination.

For Pods managed by Deployments and StatefulSets, `Always` is normally used.

Jobs generally use `OnFailure` or `Never`.

---

## 32. What is the difference between a Job retry and a container restart?

**Answer:**

A container restart happens inside an existing Pod.

A Job retry may involve creating another Pod attempt after a Pod fails, depending on the Job configuration.

### Container restart

```text
Same Pod
└── Failed container restarted
```

### Job retry

```text
Failed Pod
    |
    v
New Pod attempt
```

The Job controller tracks successful and failed completions across its attempts.

---

## 33. What are `completions` and `parallelism` in a Job?

**Answer:**

### `completions`

The number of successful Pod completions required for the Job to finish.

### `parallelism`

The maximum number of Pods that may run simultaneously for the Job.

Example:

```yaml
spec:
  completions: 10
  parallelism: 3
```

This requires ten successful completions and allows up to three active Pods at a time.

---

## 34. What is `backoffLimit` in a Job?

**Answer:**

`backoffLimit` specifies how many retries are allowed before the Job is considered failed.

Example:

```yaml
spec:
  backoffLimit: 4
```

If the Job repeatedly fails and exceeds the configured retry behavior, Kubernetes marks the Job as failed.

The exact counting behavior depends on the type of failure and Job configuration.

---

## 35. What is a ReplicaSet selector?

**Answer:**

A ReplicaSet selector identifies the Pods that belong to the ReplicaSet.

Example:

```yaml
selector:
  matchLabels:
    app: frontend
```

The Pod template must use matching labels:

```yaml
template:
  metadata:
    labels:
      app: frontend
```

If the selector and Pod template labels do not match correctly, the ReplicaSet configuration is invalid or does not manage the intended Pods.

---

## 36. Why are labels important in workloads?

**Answer:**

Labels connect Kubernetes resources.

They are used by:

- Deployments
- ReplicaSets
- Services
- NetworkPolicies
- Monitoring tools
- Scheduling rules
- Automation systems

Example:

```yaml
labels:
  app: frontend
  environment: production
```

A Service can select all Pods with:

```yaml
selector:
  app: frontend
```

---

## 37. What is the difference between a selector and a label?

**Answer:**

A label is metadata attached to an object.

A selector is a query or matching rule used to find objects with particular labels.

Example:

```text
Pod label:
app=frontend

Service selector:
app=frontend
```

The Service finds Pods whose labels match its selector.

---

## 38. What is a Pod template?

**Answer:**

A Pod template is the section of a workload resource that defines how new Pods should be created.

It commonly contains:

- Pod labels
- Container images
- Ports
- Environment variables
- Volumes
- Resource requests and limits
- Probes
- Security settings

Example:

```yaml
template:
  metadata:
    labels:
      app: frontend
  spec:
    containers:
      - name: frontend
        image: nginx:1.27
```

Deployments, ReplicaSets, StatefulSets, DaemonSets and Jobs use Pod templates.

---

## 39. What happens if you change the Pod template in a Deployment?

**Answer:**

Changing the Pod template, such as changing the image, creates a new Deployment revision.

The Deployment creates or updates a new ReplicaSet and performs a rollout according to its strategy.

Example:

```yaml
image: nginx:1.26
```

changed to:

```yaml
image: nginx:1.27
```

This normally triggers a rolling update.

Changing only the number of replicas generally scales the existing workload and does not create a new revision in the same way as a Pod template change.

---

## 40. What is a Deployment revision?

**Answer:**

A Deployment revision represents a version of the Deployment’s Pod template and rollout history.

Revisions help Kubernetes support:

- Rollout history
- Rollback
- Version tracking
- Deployment updates

Commands:

```bash
kubectl rollout history deployment/frontend
kubectl rollout undo deployment/frontend
```

---

## 41. What is a Pod Disruption Budget?

**Answer:**

A Pod Disruption Budget, or PDB, limits how many replicas of an application can be voluntarily disrupted at the same time.

Voluntary disruptions include actions such as:

- Node draining
- Cluster maintenance
- Certain administrative operations

A PDB can specify:

```yaml
minAvailable: 2
```

or:

```yaml
maxUnavailable: 1
```

A PDB does not prevent all failures. It mainly helps protect availability during voluntary disruptions.

---

## 42. What is the difference between voluntary and involuntary disruption?

**Answer:**

### Voluntary disruption

A planned or initiated action.

Examples:

- `kubectl drain`
- Node maintenance
- Cluster upgrade
- Administrative eviction

### Involuntary disruption

An unexpected failure.

Examples:

- Node crash
- Hardware failure
- Kernel panic
- Power failure
- Unexpected network failure

A PDB mainly controls voluntary disruptions and cannot guarantee protection from involuntary failures.

---

## 43. What is a workload rollout?

**Answer:**

A rollout is the process of changing the version or configuration of a workload.

For a Deployment, a rollout may occur when changing:

- Container image
- Environment variables
- Commands
- Volumes
- Pod template labels
- Resource configuration

Useful commands:

```bash
kubectl rollout status deployment/frontend
kubectl rollout history deployment/frontend
kubectl rollout pause deployment/frontend
kubectl rollout resume deployment/frontend
kubectl rollout undo deployment/frontend
```

---

## 44. What is a failed rollout?

**Answer:**

A rollout may fail or become stuck when new Pods cannot become available.

Common causes include:

- Incorrect image
- Image pull failure
- Application crash
- Failed readiness probe
- Insufficient resources
- Invalid configuration
- Missing Secret or ConfigMap
- Scheduling constraints
- Network or storage problems

Troubleshooting commands:

```bash
kubectl rollout status deployment/frontend
kubectl get pods
kubectl describe deployment frontend
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.lastTimestamp
```

---

## 45. What is `CrashLoopBackOff`?

**Answer:**

`CrashLoopBackOff` means a container is repeatedly crashing and Kubernetes is applying an increasing delay before restarting it.

Common causes:

- Application startup failure
- Incorrect command
- Missing environment variable
- Missing configuration
- Database connection failure
- Permission problem
- Application bug

Troubleshooting:

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

`--previous` is especially useful when the container has already restarted.

---

## 46. What is `ImagePullBackOff`?

**Answer:**

`ImagePullBackOff` means Kubernetes could not pull the requested image and is waiting before retrying.

Common causes:

- Incorrect image name
- Incorrect image tag
- Private registry authentication failure
- Network connectivity problem
- Registry rate limiting
- Missing imagePullSecret

Troubleshooting:

```bash
kubectl describe pod <pod-name>
kubectl get secrets
kubectl get events --sort-by=.lastTimestamp
```

---

## 47. What is a Pending Pod?

**Answer:**

A Pending Pod has not reached the Running phase.

Common reasons include:

- No suitable node
- Insufficient CPU or memory
- Untolerated taint
- Node selector mismatch
- Affinity rules
- Unbound PVC
- Volume constraints
- Scheduling restrictions

Troubleshooting:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get nodes
kubectl get pvc
```

The Events section of `kubectl describe pod` is usually very useful.

---

## 48. What is the difference between scaling and autoscaling?

**Answer:**

### Manual scaling

An administrator changes the replica count.

```bash
kubectl scale deployment frontend --replicas=5
```

### Autoscaling

Kubernetes automatically adjusts workload capacity based on metrics or policies.

Examples:

- Horizontal Pod Autoscaler
- Vertical Pod Autoscaler
- Cluster Autoscaler

Horizontal Pod Autoscaler changes the number of Pod replicas.

---

## 49. What is Horizontal Pod Autoscaler?

**Answer:**

Horizontal Pod Autoscaler, or HPA, automatically changes the number of Pod replicas based on metrics.

Metrics may include:

- CPU utilization
- Memory utilization
- Custom metrics
- External metrics

Example:

```bash
kubectl autoscale deployment frontend \
  --min=2 \
  --max=10 \
  --cpu-percent=70
```

HPA requires an appropriate metrics source, commonly Metrics Server for resource metrics.

---

## 50. What is Vertical Pod Autoscaler?

**Answer:**

Vertical Pod Autoscaler, or VPA, adjusts Pod resource requests and sometimes limits based on observed usage and configuration.

It is used to recommend or apply better CPU and memory values.

VPA and HPA can sometimes conflict if both attempt to manage the same resource dimension, so they must be designed carefully.

---

## 51. What is Cluster Autoscaler?

**Answer:**

Cluster Autoscaler adjusts the number of nodes in a cluster.

It may:

- Add nodes when Pods cannot be scheduled because of insufficient capacity
- Remove underutilized nodes when workloads can be moved safely

Cluster Autoscaler works with the infrastructure or cloud provider’s node group mechanism.

It is different from HPA:

| HPA | Cluster Autoscaler |
|---|---|
| Changes Pod count | Changes node count |
| Scales application workloads | Scales cluster infrastructure |

---

## 52. Can a Deployment run on multiple nodes?

**Answer:**

Yes.

A Deployment creates multiple Pods, and the scheduler may place them on different nodes depending on:

- Available resources
- Scheduling constraints
- Affinity and anti-affinity
- Topology spread constraints
- Taints and tolerations

To improve availability, configure anti-affinity or topology spread constraints so replicas are distributed across nodes or zones.

---

## 53. How do you ensure replicas are distributed across nodes?

**Answer:**

Use:

- Pod anti-affinity
- Topology spread constraints
- Appropriate node labels
- Multiple availability zones
- Pod Disruption Budgets

Topology spread constraints are often useful for expressing rules such as:

> Distribute frontend replicas across nodes or availability zones as evenly as possible.

---

## 54. What is the difference between a Deployment and a Job?

**Answer:**

| Deployment | Job |
|---|---|
| Runs long-lived applications | Runs finite tasks |
| Maintains desired replicas continuously | Tracks task completion |
| Usually uses `restartPolicy: Always` | Uses `OnFailure` or `Never` |
| Common for APIs and web applications | Common for migrations and batch processing |

---

## 55. What is the difference between a Job and a DaemonSet?

**Answer:**

| Job | DaemonSet |
|---|---|
| Runs a task until completion | Runs a Pod on eligible nodes |
| Usually finite execution | Usually long-running |
| Tracks successful completions | Tracks node coverage |
| Used for batch work | Used for node-level agents |

---

## 56. What is the difference between StatefulSet and DaemonSet?

**Answer:**

| StatefulSet | DaemonSet |
|---|---|
| Manages stateful application replicas | Manages node-level Pods |
| Uses stable ordinal identity | Uses node-based placement |
| Common for databases | Common for agents and plugins |
| Usually one Pod per ordinal | Usually one Pod per eligible node |

---

## 57. What is a sidecar container?

**Answer:**

A sidecar is a secondary container running in the same Pod as the main application container.

It supports the main application by providing functions such as:

- Logging
- Proxying
- Metrics collection
- Configuration reload
- Security processing

Because both containers share the Pod network namespace, they can communicate through `localhost`.

---

## 58. What is an init container?

**Answer:**

An init container runs before the application containers start.

Init containers are used for tasks such as:

- Preparing files
- Waiting for a dependency
- Performing initialization
- Downloading configuration
- Running setup scripts

Init containers run sequentially. Each must complete successfully before the next init container or application container starts.

---

## 59. Init container vs sidecar container

**Answer:**

| Init container | Sidecar container |
|---|---|
| Runs before application containers | Runs alongside application container |
| Normally completes and exits | Usually remains running |
| Used for initialization | Used for supporting the application |
| Runs in order | Runs concurrently with application containers |

---

## 60. What are lifecycle hooks?

**Answer:**

Lifecycle hooks allow actions to be executed at specific points in a container’s lifecycle.

Common hooks include:

- `postStart`
- `preStop`

`postStart` runs after a container is created, although its execution timing relative to the application process is not guaranteed.

`preStop` runs before a container is terminated and can be used for graceful shutdown preparation.

---

## 61. What is graceful termination of a Pod?

**Answer:**

When Kubernetes terminates a Pod, it normally follows a graceful shutdown process:

1. Pod termination begins.
2. The Pod is removed from relevant traffic endpoints according to readiness and endpoint processing.
3. The container runtime sends the configured termination signal, usually `SIGTERM`.
4. The application gets time to shut down.
5. After the grace period, the process may receive `SIGKILL`.

The grace period is controlled by:

```yaml
terminationGracePeriodSeconds: 30
```

Applications should handle termination signals correctly.

---

## 62. What is a termination grace period?

**Answer:**

The termination grace period is the amount of time Kubernetes gives a Pod to shut down gracefully before forceful termination.

Example:

```yaml
spec:
  terminationGracePeriodSeconds: 60
```

The application should:

- Stop accepting new work
- Finish or safely cancel active work
- Close connections
- Flush logs and data
- Exit cleanly

---

## 63. What is a PodDisruptionBudget used for during node draining?

**Answer:**

During a voluntary node drain, Kubernetes attempts to evict Pods while respecting PodDisruptionBudgets.

Example:

```yaml
minAvailable: 2
```

If evicting another Pod would violate the budget, eviction may be blocked until another replica becomes available.

A PDB does not guarantee that a workload survives node failure.

---

## 64. How do you safely update an application managed by a Deployment?

**Answer:**

A safe update process includes:

1. Use a versioned image tag.
2. Define readiness and liveness probes.
3. Configure resource requests and limits.
4. Use an appropriate rolling update strategy.
5. Set `maxUnavailable` and `maxSurge`.
6. Monitor rollout status.
7. Verify application health.
8. Roll back if necessary.

Commands:

```bash
kubectl set image deployment/frontend \
  frontend=nginx:1.27

kubectl rollout status deployment/frontend
kubectl rollout undo deployment/frontend
```

---

## 65. How do you troubleshoot a workload that is not receiving traffic?

**Answer:**

Check the following:

```bash
kubectl get pods
kubectl get svc
kubectl get endpointslices
kubectl describe svc <service-name>
kubectl describe pod <pod-name>
```

Verify:

- Pod is Running
- Pod is Ready
- Service selector matches Pod labels
- Target port is correct
- EndpointSlices contain backend addresses
- NetworkPolicy allows traffic
- Application is listening on the expected port
- Readiness probe is passing

A Pod can be running but excluded from Service traffic if it is not Ready.

---

## 66. What is the difference between readiness and liveness in workloads?

**Answer:**

### Readiness

Controls whether a Pod should receive traffic.

If readiness fails, the Pod is normally removed from Service endpoints but is not necessarily restarted.

### Liveness

Checks whether the container is unhealthy and should be restarted.

A poor liveness probe can cause unnecessary restarts. A poor readiness probe can send traffic to an application that is not ready.

---

## 67. What is a workload controller?

**Answer:**

A workload controller is a Kubernetes controller that manages Pods according to a desired behavior.

Examples:

- Deployment controller
- ReplicaSet controller
- StatefulSet controller
- DaemonSet controller
- Job controller
- CronJob controller

Controllers continuously reconcile the desired state with the actual state.

---

## 68. Why should production images use immutable or versioned tags?

**Answer:**

Using versioned or immutable image references improves:

- Reproducibility
- Rollback reliability
- Auditability
- Deployment consistency
- Troubleshooting

Using only:

```text
latest
```

can cause different nodes to pull different image contents over time.

A better approach is to use:

```text
myapp:1.4.2
```

or an image digest.

---

## 69. What is the difference between `kubectl delete pod` and deleting a Deployment?

**Answer:**

Deleting a Pod managed by a Deployment usually causes the ReplicaSet to create a replacement.

Deleting the Deployment removes the Deployment and normally its managed ReplicaSets and Pods, depending on deletion propagation behavior.

Example:

```bash
kubectl delete pod <pod-name>
```

The application usually continues running through a replacement Pod.

```bash
kubectl delete deployment frontend
```

The Deployment-managed application is normally removed.

---

## 70. Explain Kubernetes workloads in one interview answer.

**Answer:**

> Kubernetes workloads are resources used to run and manage applications or tasks. A Pod is the smallest deployable unit and contains one or more containers sharing the same network namespace and volumes. For stateless applications, a Deployment manages ReplicaSets, and ReplicaSets maintain the required number of Pods. Deployments also provide rolling updates, scaling, rollout history, and rollback.
>
> StatefulSets are used when applications need stable Pod identities and persistent storage associations. DaemonSets run a Pod on each eligible node and are commonly used for monitoring, logging, networking, and security agents. Jobs run finite tasks until completion, while CronJobs create Jobs on a schedule.
>
> Kubernetes controllers continuously compare desired state with actual state. If a Pod crashes or is deleted, the appropriate controller attempts to restore the required state. Workload reliability is improved using readiness probes, liveness probes, resource requests and limits, Pod Disruption Budgets, rolling update strategies, and proper scheduling rules.
