# Kubernetes Scheduling Interview Questions and Answers

## 1. What is Kubernetes Scheduling?

Kubernetes scheduling is the process of selecting a suitable node for a newly created Pod.

The Kubernetes Scheduler watches for Pods that do not yet have a node assigned. It evaluates available nodes based on:

- CPU and memory requests
- Node readiness
- Resource availability
- Node selectors
- Node affinity
- Pod affinity and anti-affinity
- Taints and tolerations
- Topology constraints
- Scheduling policies
- Priority and preemption

After selecting a node, the Scheduler binds the Pod to that node. The kubelet on that node then creates and runs the containers.

---

## 2. What is the role of the Kubernetes Scheduler?

The Scheduler assigns unscheduled Pods to appropriate nodes.

Its basic responsibilities are:

1. Watch for unscheduled Pods.
2. Filter nodes that cannot run the Pod.
3. Score feasible nodes.
4. Select the best node.
5. Bind the Pod to the selected node.

The Scheduler does not normally start containers directly. The kubelet performs the actual Pod execution on the assigned node.

---

## 3. What is an unscheduled Pod?

An unscheduled Pod is a Pod whose `spec.nodeName` has not been assigned.

Check with:

```bash
kubectl get pods -o wide
```

If a Pod remains in `Pending`, possible reasons include:

- No node has enough resources.
- Node selector does not match any node.
- Affinity rules cannot be satisfied.
- A taint is not tolerated.
- Persistent volume topology is incompatible.
- Pod quota or scheduling constraints block placement.

Use:

```bash
kubectl describe pod <pod-name>
```

The Events section usually shows the reason.

---

## 4. What is the difference between Scheduler and kubelet?

| Scheduler | Kubelet |
|---|---|
| Selects a node for a Pod | Runs the Pod on its node |
| Cluster-level control-plane component | Node-level agent |
| Evaluates scheduling constraints | Creates and monitors containers |
| Binds Pod to a node | Reports Pod status |
| Does not normally run containers | Uses container runtime to run containers |

Flow:

```text
Pod created
   |
Scheduler selects node
   |
Pod assigned to node
   |
Kubelet starts containers
```

---

## 5. What are the main stages of scheduling?

A simplified scheduling cycle includes:

### 1. Scheduling queue

Pods waiting to be scheduled are placed in queues.

### 2. Filtering

The Scheduler removes nodes that cannot run the Pod.

### 3. Scoring

The Scheduler ranks the remaining feasible nodes.

### 4. Binding

The selected node is assigned to the Pod.

The Scheduler may also use reserve, permit, pre-bind, and post-bind extension points through scheduling plugins.

---

## 6. What is filtering in Kubernetes scheduling?

Filtering identifies nodes that are not suitable for a Pod.

Examples of filter checks:

- Node has insufficient CPU.
- Node has insufficient memory.
- Node is not Ready.
- Node selector does not match.
- Required node affinity fails.
- Pod does not tolerate a taint.
- Volume cannot be used on the node.
- Pod topology constraints cannot be satisfied.

Only nodes that pass filtering are considered for scoring.

---

## 7. What is scoring in Kubernetes scheduling?

Scoring ranks nodes that passed filtering.

The Scheduler assigns scores based on configured plugins and preferences.

Examples:

- Prefer spreading Pods across zones.
- Prefer nodes with more available resources.
- Prefer nodes matching preferred affinity.
- Prefer balanced resource utilization.
- Prefer fewer conflicting Pods.

The highest-scoring feasible node is normally selected.

---

## 8. What is the difference between filtering and scoring?

| Filtering | Scoring |
|---|---|
| Removes unsuitable nodes | Ranks suitable nodes |
| Uses hard requirements | Uses preferences |
| A failed filter excludes a node | A low score does not necessarily exclude a node |
| Example: required node affinity | Example: preferred node affinity |

Interview example:

- `requiredDuringSchedulingIgnoredDuringExecution` is a hard constraint.
- `preferredDuringSchedulingIgnoredDuringExecution` is a soft preference.

---

## 9. What are CPU and memory requests?

Resource requests specify the amount of CPU and memory a container needs for scheduling.

Example:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

The Scheduler uses requests—not current real-time usage—to determine whether a node has enough allocatable capacity.

Meaning:

- `500m` CPU = 0.5 CPU core.
- `512Mi` = 512 mebibytes of memory.

Requests are also used by Kubernetes for resource accounting and QoS classification.

---

## 10. What are CPU and memory limits?

Limits define the maximum amount of a resource a container may use, subject to runtime and resource behavior.

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: "1Gi"
```

Scheduling primarily uses requests. Limits are important for runtime enforcement but do not directly represent the amount of capacity reserved for scheduling.

A Pod with high limits but low requests may be scheduled based on the lower requests.

---

## 11. What happens if a Pod has no resource requests?

If a Pod has no requests:

- The Scheduler may treat the request as zero for scheduling purposes.
- The Pod can be placed on a node with little remaining capacity.
- It may receive a lower QoS classification.
- It can contribute to resource contention.
- It may be evicted under node pressure.

Best practice: define realistic CPU and memory requests for production workloads.

---

## 12. What is node allocatable?

Node allocatable is the amount of node resources available for Pods after reserving resources for system components and Kubernetes services.

A node’s total capacity is not necessarily fully available to workloads.

View it with:

```bash
kubectl describe node <node-name>
```

Look for:

```text
Capacity:
Allocatable:
```

Allocatable accounts for reservations such as:

- System processes
- Kubernetes system daemons
- Eviction thresholds
- Other configured reservations

---

## 13. What is `nodeSelector`?

`nodeSelector` is the simplest way to constrain a Pod to nodes with specific labels.

Example:

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

The Pod can run only on nodes labeled:

```bash
kubectl label node worker-1 disktype=ssd
```

`nodeSelector` is a hard requirement. If no node matches, the Pod remains unscheduled.

---

## 14. What is node affinity?

Node affinity is a more expressive alternative to `nodeSelector`.

It supports:

- Required rules
- Preferred rules
- Multiple expressions
- Operators such as `In`, `NotIn`, `Exists`, and `DoesNotExist`

Example:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

---

## 15. What is the difference between `nodeSelector` and node affinity?

| `nodeSelector` | Node affinity |
|---|---|
| Simple key-value matching | Expressive rule syntax |
| Hard constraint | Can be hard or preferred |
| Limited operators | Supports multiple operators |
| Easy for basic placement | Better for complex scheduling policies |

Use `nodeSelector` for simple rules and node affinity for advanced placement logic.

---

## 16. What does `requiredDuringSchedulingIgnoredDuringExecution` mean?

This is a required node-affinity rule.

Meaning:

- `requiredDuringScheduling`: the rule must be satisfied when the Pod is scheduled.
- `IgnoredDuringExecution`: if node labels later change, Kubernetes does not automatically evict the already-running Pod because of that change.

Example:

```yaml
requiredDuringSchedulingIgnoredDuringExecution:
  nodeSelectorTerms:
    - matchExpressions:
        - key: environment
          operator: In
          values:
            - production
```

The Pod cannot initially schedule onto a node that does not match.

---

## 17. What does `preferredDuringSchedulingIgnoredDuringExecution` mean?

This is a preferred node-affinity rule.

Meaning:

- The Scheduler tries to place the Pod on a matching node.
- It may still schedule the Pod elsewhere if no preferred node is available.
- The preference is used during scoring.
- Label changes after scheduling do not automatically evict the Pod.

Example:

```yaml
preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 80
    preference:
      matchExpressions:
        - key: disk
          operator: In
          values:
            - ssd
```

The weight influences the preference score.

---

## 18. What is Pod affinity?

Pod affinity places a Pod near other Pods based on labels and topology domains.

Example use cases:

- Application Pod near its cache.
- Frontend Pod near backend Pod.
- Components that communicate frequently.

Example:

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - cache
        topologyKey: kubernetes.io/hostname
```

This requires the Pod to be scheduled in the same topology domain as matching Pods.

---

## 19. What is Pod anti-affinity?

Pod anti-affinity prevents or discourages placing a Pod near other matching Pods.

Common use case:

- Spread replicas across nodes.
- Avoid placing all database replicas on one node.
- Improve availability during node failure.

Example:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: database
        topologyKey: kubernetes.io/hostname
```

This prevents matching Pods from sharing the same hostname topology domain.

---

## 20. What is the difference between Pod affinity and node affinity?

| Node affinity | Pod affinity |
|---|---|
| Matches node labels | Matches labels of other Pods |
| Places Pod on suitable nodes | Places Pod relative to other Pods |
| Example: SSD nodes | Example: same node as cache |
| Uses node topology | Uses Pod placement and topology |

Pod affinity and node affinity can be combined in the same Pod specification.

---

## 21. What is required versus preferred affinity?

### Required affinity

The rule must be satisfied.

If no node satisfies it, the Pod remains pending.

### Preferred affinity

The Scheduler tries to satisfy the rule but may ignore it if necessary.

Required rules are filters. Preferred rules influence scoring.

---

## 22. What is `topologyKey`?

`topologyKey` identifies the topology domain used for affinity, anti-affinity, and topology spread rules.

Common labels include:

```text
kubernetes.io/hostname
topology.kubernetes.io/zone
topology.kubernetes.io/region
```

Examples:

- Hostname = spread across nodes.
- Zone = spread across availability zones.
- Region = spread across regions.

The label must exist consistently on the relevant nodes.

---

## 23. What are topology spread constraints?

Topology spread constraints control how Pods are distributed across topology domains.

They help achieve:

- High availability
- Even distribution
- Zone balancing
- Node balancing
- Reduced concentration of replicas

Example:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: web
```

---

## 24. What is `maxSkew`?

`maxSkew` is the maximum permitted difference in the number of matching Pods between topology domains, subject to the applicable constraint rules.

Example:

```yaml
maxSkew: 1
```

If one zone has 3 matching Pods and another has 1, the skew is 2. A constraint with `maxSkew: 1` may prevent further placement that increases the imbalance.

`maxSkew` must be understood together with:

- `topologyKey`
- `whenUnsatisfiable`
- `labelSelector`
- Eligible domains
- Node labels

---

## 25. What is `whenUnsatisfiable`?

This field defines what happens when a topology spread constraint cannot be satisfied.

Values:

- `DoNotSchedule`: treat the constraint as a hard requirement.
- `ScheduleAnyway`: schedule the Pod but prioritize nodes that reduce skew.

Example:

```yaml
whenUnsatisfiable: DoNotSchedule
```

Use `DoNotSchedule` for strict distribution and `ScheduleAnyway` when availability is more important than perfect spreading.

---

## 26. What are taints?

A taint is applied to a node to repel Pods that do not tolerate it.

Example:

```bash
kubectl taint nodes worker-1 dedicated=database:NoSchedule
```

This means Pods without a matching toleration should not be scheduled on `worker-1`.

Taints are commonly used for:

- Dedicated nodes
- GPU nodes
- Control-plane nodes
- Special hardware
- Maintenance or isolation

---

## 27. What are tolerations?

A toleration allows a Pod to be considered for scheduling onto a node with a matching taint.

Example:

```yaml
tolerations:
  - key: dedicated
    operator: Equal
    value: database
    effect: NoSchedule
```

A toleration does not force the Pod onto that node. It only removes the taint-based scheduling barrier.

To force placement, combine tolerations with node affinity or node selectors.

---

## 28. What are the taint effects?

The main taint effects are:

### `NoSchedule`

New Pods without a matching toleration are not scheduled onto the node.

### `PreferNoSchedule`

The Scheduler tries to avoid placing non-tolerating Pods there, but it is a soft preference.

### `NoExecute`

- Prevents new non-tolerating Pods from scheduling.
- Can evict already-running non-tolerating Pods.

Example:

```bash
kubectl taint nodes worker-1 maintenance=true:NoExecute
```

---

## 29. What is the difference between `NoSchedule` and `NoExecute`?

| Effect | New Pods | Existing Pods |
|---|---|---|
| `NoSchedule` | Blocks non-tolerating Pods | Existing Pods usually remain |
| `PreferNoSchedule` | Tries to avoid placement | Existing Pods remain |
| `NoExecute` | Blocks non-tolerating Pods | Can evict non-tolerating Pods |

A toleration can include `tolerationSeconds` for a `NoExecute` taint.

---

## 30. What is `tolerationSeconds`?

`tolerationSeconds` specifies how long a Pod may remain bound to a node after a matching `NoExecute` taint is applied.

Example:

```yaml
tolerations:
  - key: maintenance
    operator: Equal
    value: planned
    effect: NoExecute
    tolerationSeconds: 300
```

The Pod may remain for approximately 300 seconds before eviction, subject to the taint and controller behavior.

---

## 31. What is the difference between taints and node affinity?

| Taints and tolerations | Node affinity |
|---|---|
| Repel Pods from nodes | Attract or require Pods on nodes |
| Configured on nodes and Pods | Configured mainly on Pods |
| Toleration permits scheduling | Affinity selects matching nodes |
| Useful for dedicated or restricted nodes | Useful for placement rules |

For dedicated nodes, use both:

1. Taint the node.
2. Add a matching toleration.
3. Add node affinity if only selected workloads should use it.

---

## 32. What is a dedicated node?

A dedicated node is reserved for a particular workload or workload class.

Example:

```bash
kubectl taint nodes worker-1 workload=database:NoSchedule
kubectl label node worker-1 workload=database
```

Pod configuration:

```yaml
tolerations:
  - key: workload
    operator: Equal
    value: database
    effect: NoSchedule

nodeSelector:
  workload: database
```

The toleration allows access, while the selector directs the Pod to the dedicated node.

---

## 33. What is cordoning a node?

Cordoning marks a node as unschedulable for new Pods.

Command:

```bash
kubectl cordon <node-name>
```

Effects:

- New ordinary Pods are not scheduled there.
- Existing Pods continue running.
- DaemonSet behavior and special cases may differ.

Check:

```bash
kubectl get nodes
```

A cordoned node usually shows `SchedulingDisabled`.

---

## 34. What is draining a node?

Draining a node evicts eligible Pods so that the node can be maintained.

Command:

```bash
kubectl drain <node-name> --ignore-daemonsets
```

Drain may require additional options for unmanaged Pods or local storage.

Example:

```bash
kubectl drain worker-1 \
  --ignore-daemonsets \
  --delete-emptydir-data
```

Drain respects PodDisruptionBudgets where applicable and may fail if disruption is not allowed.

---

## 35. What is the difference between cordon and drain?

| Cordon | Drain |
|---|---|
| Stops new scheduling | Evicts eligible existing Pods |
| Existing Pods remain | Existing Pods are moved or recreated |
| Quick scheduling control | Used for maintenance |
| Does not delete workloads | May terminate Pods |

Typical maintenance flow:

```bash
kubectl cordon worker-1
kubectl drain worker-1 --ignore-daemonsets
# perform maintenance
kubectl uncordon worker-1
```

---

## 36. What is `unschedulable` in a Node?

A node with `spec.unschedulable: true` is marked unschedulable.

This is commonly set by:

```bash
kubectl cordon <node-name>
```

It prevents normal scheduling of new Pods but does not automatically terminate existing Pods.

---

## 37. What is node readiness?

A node’s Ready condition indicates whether it is healthy enough to accept workloads.

Check:

```bash
kubectl get nodes
kubectl describe node <node-name>
```

Possible node conditions include:

- `Ready`
- `MemoryPressure`
- `DiskPressure`
- `PIDPressure`
- `NetworkUnavailable`

A node under pressure may become unsuitable for new scheduling or may trigger Pod eviction.

---

## 38. What is node pressure?

Node pressure occurs when a node lacks important resources.

Examples:

- Memory pressure
- Disk pressure
- PID pressure

Kubernetes may:

- Mark node conditions.
- Trigger eviction.
- Prevent scheduling.
- Remove or deprioritize workloads.
- Require administrator intervention.

Check:

```bash
kubectl describe node <node-name>
kubectl get events
```

---

## 39. What is Pod priority?

Pod priority determines the relative importance of Pods during scheduling and preemption.

A higher-priority Pod may be scheduled before lower-priority Pods.

Priority is defined through a `PriorityClass`.

Example:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "High priority workloads"
```

Pod reference:

```yaml
priorityClassName: high-priority
```

---

## 40. What is preemption?

Preemption allows the Scheduler to evict lower-priority Pods to make room for a higher-priority Pod when no suitable node has enough capacity.

Simplified flow:

1. High-priority Pod cannot fit on any node.
2. Scheduler identifies lower-priority victims.
3. Victim Pods are selected for removal.
4. High-priority Pod can be scheduled after resources become available.

Preemption should be used carefully because it can disrupt running workloads.

---

## 41. What is the difference between priority and preemption?

| Priority | Preemption |
|---|---|
| Assigns importance to Pods | Evicts lower-priority Pods when necessary |
| Affects scheduling order | Frees capacity for higher-priority Pods |
| Does not always evict Pods | Can disrupt existing workloads |
| Defined using PriorityClass | Uses priority during scheduling decisions |

A high-priority Pod does not automatically preempt another Pod if it can be scheduled without preemption.

---

## 42. What is a `PriorityClass`?

A PriorityClass is a cluster-scoped object that assigns a priority value to Pods.

Example:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-workload
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
```

Important fields:

- `value`
- `globalDefault`
- `description`
- `preemptionPolicy`

Higher values represent higher priority.

---

## 43. What is `preemptionPolicy: Never`?

`preemptionPolicy: Never` allows a Pod to have high scheduling priority without evicting lower-priority Pods.

Such Pods may jump ahead in the scheduling queue but cannot preempt running workloads.

Example:

```yaml
preemptionPolicy: Never
```

This is useful for important batch jobs that should be scheduled ahead of other queued work but should not disrupt running applications.

---

## 44. What is a static Pod?

A static Pod is managed directly by the kubelet on a specific node rather than by the Kubernetes Scheduler.

Static Pod manifests are commonly stored in a directory configured for the kubelet, such as:

```text
/etc/kubernetes/manifests/
```

Examples in kubeadm clusters include control-plane components.

Static Pods:

- Are tied to one node.
- Are not scheduled by the normal Scheduler.
- Are monitored by the kubelet.
- May have mirror Pods visible through the API server.

---

## 45. What is the difference between a static Pod and a normal Pod?

| Static Pod | Normal Pod |
|---|---|
| Managed by kubelet | Usually managed through API server/controllers |
| Runs on a specific node | Scheduler selects a node |
| Manifest exists on node | Object usually created through API |
| Not scheduled normally | Scheduled by Scheduler |
| Often used for node-level control-plane components | Used for application workloads |

---

## 46. What is `nodeName`?

`spec.nodeName` directly assigns a Pod to a specific node.

Example:

```yaml
spec:
  nodeName: worker-1
```

When `nodeName` is set, the normal Scheduler is bypassed.

This should be used carefully because it bypasses normal scheduling checks and can cause the Pod to be assigned to an unsuitable node.

---

## 47. What is the difference between `nodeName` and `nodeSelector`?

| `nodeName` | `nodeSelector` |
|---|---|
| Directly names a node | Selects nodes by labels |
| Bypasses normal scheduling | Uses Scheduler |
| Tightly couples Pod to node | More flexible |
| Useful for special cases | Preferred for normal placement rules |

Use node affinity or node selectors for most application scheduling requirements.

---

## 48. What is schedulerName?

`spec.schedulerName` tells Kubernetes which scheduler should handle a Pod.

Example:

```yaml
spec:
  schedulerName: custom-scheduler
```

If omitted, the default scheduler name is generally `default-scheduler`.

A custom scheduler must be deployed and configured to watch and schedule the relevant Pods.

---

## 49. Can Kubernetes have multiple schedulers?

Yes. Kubernetes can run multiple schedulers.

Pods select a scheduler using:

```yaml
spec:
  schedulerName: custom-scheduler
```

Use cases:

- Specialized hardware placement
- Custom scheduling policies
- Research or experimentation
- Workload-specific scheduling

The scheduler must be correctly configured and must not conflict with other schedulers.

---

## 50. What is the scheduling framework?

The scheduling framework provides extension points for scheduler plugins.

Plugins can participate in stages such as:

- QueueSort
- PreFilter
- Filter
- PostFilter
- PreScore
- Score
- Reserve
- Permit
- PreBind
- Bind
- PostBind

This allows scheduling behavior to be extended without replacing the entire Scheduler.

---

## 51. What is a scheduling profile?

A scheduling profile configures the plugins and behavior used by a scheduler.

Multiple profiles can be configured in one scheduler process, with each profile selected through a scheduler name.

Profiles can support different scheduling policies for different workloads.

---

## 52. What is the role of the scheduler extender?

A scheduler extender is an external HTTP-based extension that can influence scheduling decisions.

It may participate in operations such as:

- Filtering nodes
- Prioritizing nodes
- Binding Pods

Modern scheduling designs often prefer the scheduling framework and plugins, but extenders may still exist in some environments.

---

## 53. What is the difference between default scheduler and custom scheduler?

| Default scheduler | Custom scheduler |
|---|---|
| Provided by Kubernetes | Built or configured for special requirements |
| Uses standard scheduling plugins | Can use custom policies |
| Handles Pods using default scheduler name | Handles Pods assigned to its scheduler name |
| Suitable for most workloads | Useful for specialized scheduling |

---

## 54. What is scheduler latency?

Scheduler latency is the time taken to schedule a Pod after it becomes eligible.

It may increase because of:

- Large cluster size
- Many pending Pods
- Complex affinity rules
- Expensive custom plugins
- API server latency
- Resource contention
- Large numbers of nodes or topology domains

Monitor scheduler metrics and inspect pending Pod events when diagnosing delays.

---

## 55. What is a scheduling failure?

A scheduling failure occurs when the Scheduler cannot find a feasible node for a Pod.

Common messages include:

```text
0/3 nodes are available:
2 Insufficient cpu,
1 node(s) had taint ...
```

Typical causes:

- Insufficient resources
- Node affinity mismatch
- Untolerated taint
- Volume topology conflict
- Node unschedulable
- Pod anti-affinity conflict
- Topology spread constraint
- Resource quota or admission restrictions

---

## 56. How do you troubleshoot a Pod stuck in Pending?

Use:

```bash
kubectl get pod <pod-name>
kubectl describe pod <pod-name>
kubectl get nodes
kubectl describe nodes
kubectl get events --sort-by=.lastTimestamp
```

Check:

1. Scheduler Events.
2. CPU and memory requests.
3. Node allocatable capacity.
4. Node selectors and affinity.
5. Taints and tolerations.
6. Pod anti-affinity.
7. Topology spread constraints.
8. PVC and storage topology.
9. Resource quotas.
10. Priority and preemption behavior.

The `FailedScheduling` event is usually the starting point.

---

## 57. How do you check node labels?

Commands:

```bash
kubectl get nodes --show-labels
kubectl get node <node-name> --show-labels
kubectl describe node <node-name>
```

Add a label:

```bash
kubectl label node worker-1 disktype=ssd
```

Remove a label:

```bash
kubectl label node worker-1 disktype-
```

Be careful when changing labels used by required affinity rules.

---

## 58. How do you check node taints?

Commands:

```bash
kubectl describe node <node-name>
kubectl get node <node-name> -o jsonpath='{.spec.taints}'
```

Add a taint:

```bash
kubectl taint nodes worker-1 dedicated=database:NoSchedule
```

Remove a taint:

```bash
kubectl taint nodes worker-1 dedicated=database:NoSchedule-
```

A Pod needs a matching toleration to pass the taint filter.

---

## 59. How do you check resource availability on nodes?

Commands:

```bash
kubectl describe node <node-name>
kubectl top nodes
kubectl get nodes -o wide
```

`kubectl describe node` shows capacity and allocatable resources.

`kubectl top nodes` shows current usage if Metrics Server is installed.

Remember: scheduling is based mainly on requests and allocatable resources, not only current usage.

---

## 60. How do you troubleshoot insufficient CPU or memory?

Check:

```bash
kubectl describe pod <pod-name>
kubectl describe nodes
kubectl get pods -A -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,CPU:.spec.containers[*].resources.requests.cpu,MEMORY:.spec.containers[*].resources.requests.memory
```

Possible solutions:

- Reduce unrealistic requests.
- Add worker nodes.
- Scale the cluster.
- Use a different node pool.
- Adjust workload distribution.
- Review namespace quotas.
- Check for stuck or terminating Pods.
- Use appropriate autoscaling.

Do not simply reduce requests below the application’s real needs.

---

## 61. How do taints cause a Pending Pod?

Suppose a node has:

```text
dedicated=database:NoSchedule
```

A Pod without the matching toleration cannot be scheduled there.

The event may contain:

```text
had untolerated taint
```

Fix by either:

- Adding a correct toleration.
- Removing or changing the taint if it is unnecessary.
- Scheduling the Pod on another suitable node.

A toleration alone does not guarantee placement.

---

## 62. How does node affinity cause a Pending Pod?

A required node affinity rule may require:

```text
environment=production
```

If no node has that label, the Pod remains Pending.

Check:

```bash
kubectl get nodes --show-labels
kubectl describe pod <pod-name>
```

Fix:

- Add the required label to eligible nodes.
- Correct the affinity key/value.
- Change required affinity to preferred if strict placement is not necessary.

---

## 63. How does Pod anti-affinity cause scheduling failure?

Required Pod anti-affinity can prevent a new replica from being placed near existing matching Pods.

Example:

- Three replicas require one Pod per node.
- Only two suitable nodes exist.
- The third replica cannot be scheduled.

The Pod may remain Pending until:

- Another node becomes available.
- The anti-affinity rule is relaxed.
- The workload is scaled down.
- Topology or labels are corrected.

---

## 64. How does a PVC affect scheduling?

A PVC can affect scheduling when:

- The volume is restricted to a zone.
- The CSI driver requires a particular topology.
- The volume supports only one node.
- The claim is not yet bound.
- The StorageClass uses `WaitForFirstConsumer`.

The Scheduler must choose a node compatible with both the Pod and its storage requirements.

Check:

```bash
kubectl describe pvc <pvc-name>
kubectl get storageclass
kubectl describe pod <pod-name>
```

---

## 65. What is scheduling with GPUs?

GPU workloads usually require:

- GPU device plugin
- Node labels
- Resource requests such as `nvidia.com/gpu`
- Appropriate taints and tolerations
- Compatible drivers

Example:

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

A GPU resource is generally requested through limits, and the device plugin advertises the resource to Kubernetes.

GPU nodes are often tainted to prevent ordinary workloads from consuming them.

---

## 66. What is scheduling with extended resources?

Extended resources are custom node resources advertised by device plugins or other integrations.

Examples:

```text
nvidia.com/gpu
example.com/fpga
```

A Pod requests them through its resource specification:

```yaml
resources:
  limits:
    example.com/fpga: 1
```

The Scheduler considers the advertised resource while selecting a node.

---

## 67. What is the relationship between HPA, Cluster Autoscaler, and Scheduler?

### HPA

Changes the number of Pod replicas based on metrics.

### Scheduler

Places newly created Pods onto suitable nodes.

### Cluster Autoscaler

Adds or removes nodes based on cluster capacity and utilization, including unschedulable Pods.

Typical flow:

```text
HPA increases replicas
        |
New Pods created
        |
Scheduler cannot place some Pods
        |
Cluster Autoscaler adds nodes
        |
Scheduler places pending Pods
```

These components solve different problems.

---

## 68. What is the difference between DaemonSet scheduling and normal scheduling?

DaemonSets ensure that a Pod runs on eligible nodes.

The DaemonSet controller creates Pods for nodes according to its rules. DaemonSet Pods may use the default Scheduler, but DaemonSet behavior includes special handling for node targeting and scheduling.

Common use cases:

- Node Exporter
- Log agents
- CNI components
- Storage node plugins

DaemonSets may tolerate common node taints depending on their configuration.

---

## 69. What are Kubernetes scheduling best practices?

1. Define realistic CPU and memory requests.
2. Use node labels for clear placement rules.
3. Prefer node affinity for complex constraints.
4. Use taints and tolerations for dedicated nodes.
5. Use topology spread constraints for availability.
6. Avoid excessive required anti-affinity.
7. Use `WaitForFirstConsumer` for topology-aware storage.
8. Monitor Pending Pods and `FailedScheduling` events.
9. Use Pod priority carefully.
10. Avoid unnecessary direct `nodeName` assignments.
11. Keep node labels consistent.
12. Use autoscaling where appropriate.
13. Test scheduling rules in a non-production namespace.
14. Document dedicated node pools and taints.
15. Balance availability, cost, and resource utilization.

---

## 70. Explain Kubernetes Scheduling in an interview-ready answer.

Kubernetes Scheduling is the process of assigning unscheduled Pods to suitable nodes. The Scheduler first filters nodes that cannot run the Pod because of insufficient resources, node selectors, required affinity, taints, volume constraints, or topology rules. It then scores the feasible nodes based on preferences such as preferred affinity, resource balance, and workload distribution, and binds the Pod to the selected node.

Scheduling decisions primarily use CPU and memory requests against node allocatable capacity. Node selectors and node affinity control placement based on node labels, while Pod affinity and anti-affinity control placement relative to other Pods. Taints repel workloads unless they have matching tolerations, and topology spread constraints distribute replicas across nodes or zones.

Priority and preemption allow important Pods to be scheduled ahead of lower-priority workloads, sometimes by evicting lower-priority Pods. Operationally, Pending Pods should be diagnosed using `kubectl describe pod`, node capacity, labels, taints, affinity rules, topology constraints, storage requirements, and scheduler Events. The Scheduler places Pods; the kubelet actually runs them.
