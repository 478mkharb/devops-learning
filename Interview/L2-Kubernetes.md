# Kubernetes Senior L2 Interview Preparation

## How to Use This Guide

This README is designed for **Senior L2 Kubernetes / DevOps
interviews**.

Focus on: - Troubleshooting methodology - Kubernetes networking -
Scheduling - Storage - Deployments and rollouts - Probes and application
failures - Security and RBAC - Production debugging - Cluster
components - Practical `kubectl` commands

A strong interview answer should normally follow:

**Symptom → Commands → Evidence → Root Cause → Fix → Prevention**

------------------------------------------------------------------------

# 1. Kubernetes Architecture

## Q1. Explain the Kubernetes architecture.

### Answer

A Kubernetes cluster has two major parts:

**Control Plane** - kube-apiserver - etcd - kube-scheduler -
kube-controller-manager - cloud-controller-manager, when applicable

**Worker Nodes** - kubelet - kube-proxy - container runtime such as
containerd - Pods

Typical flow:

``` text
kubectl
   |
   v
kube-apiserver
   |
   +----> etcd
   |
   +----> scheduler
   |
   +----> controller-manager
   |
   v
kubelet on worker
   |
   v
container runtime
   |
   v
Pod
```

The **API server is the central entry point** for Kubernetes API
operations.

`etcd` stores Kubernetes cluster state.

The scheduler decides which node should run a newly created Pod.

Controllers continuously compare the desired state with the actual state
and take corrective action.

------------------------------------------------------------------------

## Q2. What happens internally when you run `kubectl apply -f deployment.yaml`?

### Answer

1.  `kubectl` authenticates with the kube-apiserver.
2.  The API server authenticates and authorizes the request.
3.  Admission controls may validate or mutate the object.
4.  The Deployment object is persisted in etcd.
5.  The Deployment controller detects the desired Deployment.
6.  It creates/updates a ReplicaSet.
7.  The ReplicaSet controller creates Pods.
8.  The scheduler watches for unscheduled Pods.
9.  The scheduler selects a suitable node.
10. The kubelet on that node sees the assigned Pod.
11. kubelet asks the container runtime to create the containers.
12. CNI configures Pod networking.
13. The containers start.
14. kubelet continuously reports Pod/node status to the API server.

------------------------------------------------------------------------

# 2. Pod Troubleshooting

## Q3. A Pod is stuck in `Pending`. How do you troubleshoot it?

### Answer

Start with:

``` bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

The most important place is:

``` text
Events:
```

Then check:

``` bash
kubectl get nodes
kubectl describe nodes
```

Possible causes:

-   Insufficient CPU
-   Insufficient memory
-   Node has a taint
-   Pod has no matching toleration
-   NodeSelector mismatch
-   Node affinity mismatch
-   Pod anti-affinity constraints
-   ResourceQuota
-   LimitRange
-   PVC not bound
-   Unschedulable nodes

Check resources:

``` bash
kubectl describe node <node-name>
kubectl top nodes
kubectl top pods -A
```

For PVC:

``` bash
kubectl get pvc -A
kubectl describe pvc <pvc-name>
```

The **Pod events and scheduler messages** normally reveal why the Pod
cannot be scheduled.

------------------------------------------------------------------------

## Q4. A Pod is in `CrashLoopBackOff`. What do you check?

### Answer

First:

``` bash
kubectl get pod <pod-name>
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

If the container restarts quickly:

``` bash
kubectl logs <pod-name> --previous
```

Check:

``` bash
kubectl get pod <pod-name> -o wide
kubectl describe pod <pod-name>
```

I would investigate:

-   Application crash
-   Incorrect environment variables
-   Missing ConfigMap/Secret
-   Wrong command/entrypoint
-   Dependency unavailable
-   Database connection failure
-   Failed liveness probe
-   OOMKilled
-   Permission problems
-   Configuration errors

For OOM:

``` bash
kubectl describe pod <pod-name>
```

Look for:

``` text
Reason: OOMKilled
Exit Code: 137
```

`--previous` is particularly useful because the current container may
already have restarted.

------------------------------------------------------------------------

## Q5. A Pod is `Running`, but the application is not accessible. What do you check?

### Answer

`Running` only means the containers are running. It does not guarantee
that the application is healthy or reachable.

I would check:

``` bash
kubectl get pod <pod-name> -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Then verify the application from inside the Pod:

``` bash
kubectl exec -it <pod-name> -- curl localhost:<port>
```

Check the Service:

``` bash
kubectl get svc
kubectl describe svc <service-name>
```

Check endpoints:

``` bash
kubectl get endpoints <service-name>
kubectl get endpointslices
```

If there are no endpoints, I check whether the Service selector matches
Pod labels.

Then check:

``` bash
kubectl get pods --show-labels
kubectl get svc <service-name> -o yaml
```

For external access, investigate:

``` text
Ingress
    |
Service
    |
Endpoints
    |
Pods
```

------------------------------------------------------------------------

# 3. Kubernetes Services and Networking

## Q6. What are the main Kubernetes Service types?

### Answer

The common Service types are:

1.  `ClusterIP`
2.  `NodePort`
3.  `LoadBalancer`
4.  `ExternalName`

### ClusterIP

Default Service type.

Used for internal communication inside the cluster.

``` text
Pod A ---> ClusterIP Service ---> Pod B
```

### NodePort

Exposes a Service through a port on every node.

``` text
Client
  |
NodeIP:NodePort
  |
Service
  |
Pod
```

### LoadBalancer

Usually provisions or integrates with an external cloud load balancer.

``` text
Internet
   |
Cloud Load Balancer
   |
Service
   |
Pods
```

### ExternalName

Maps a Service name to an external DNS name using a CNAME.

------------------------------------------------------------------------

## Q7. Explain the traffic flow from a user to a Kubernetes application.

### Answer

A common cloud architecture is:

``` text
User
 |
DNS
 |
Load Balancer
 |
Ingress Controller
 |
Ingress
 |
Service
 |
Endpoints / EndpointSlices
 |
Pod
 |
Container
```

For a `LoadBalancer` Service, the exact implementation depends on the
cloud provider and networking implementation.

The Service selects Pods using labels.

For example:

``` yaml
selector:
  app: frontend
```

The selected Pods become Service endpoints.

------------------------------------------------------------------------

## Q8. A Service exists, but `kubectl get endpoints <service>` shows `<none>`. What is the problem?

### Answer

Most commonly, the Service selector does not match any Pod labels.

I would compare:

``` bash
kubectl get svc <service-name> -o yaml
kubectl get pods --show-labels
```

For example:

Service:

``` yaml
selector:
  app: frontend
```

Pod:

``` yaml
labels:
  app: backend
```

There is no match, so the Service has no endpoints.

I would also check:

``` bash
kubectl get endpointslices
```

And verify that the selected Pods are Ready.

------------------------------------------------------------------------

## Q9. What is the difference between Endpoints and EndpointSlices?

### Answer

`Endpoints` is the older API representation.

`EndpointSlice` is the newer and more scalable mechanism for
representing Service endpoints.

EndpointSlices divide endpoints into smaller objects instead of putting
all endpoints into one large object.

Useful commands:

``` bash
kubectl get endpoints <service>
kubectl get endpointslices
kubectl describe endpointslice <name>
```

------------------------------------------------------------------------

# 4. DNS

## Q10. How does DNS work inside Kubernetes?

### Answer

Kubernetes normally uses CoreDNS for cluster DNS.

A Pod can access a Service using:

``` text
<service-name>
```

within the same namespace.

For another namespace:

``` text
<service-name>.<namespace>
```

The fully qualified form is approximately:

``` text
<service-name>.<namespace>.svc.cluster.local
```

Example:

``` bash
curl http://employee-api.otms.svc.cluster.local:8080
```

Troubleshooting:

``` bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

From a Pod:

``` bash
kubectl exec -it <pod> -- nslookup employee-api
```

------------------------------------------------------------------------

# 5. ConfigMaps and Secrets

## Q11. What is the difference between ConfigMap and Secret?

### Answer

`ConfigMap` stores non-sensitive configuration.

Examples:

-   Application configuration
-   URLs
-   Feature flags
-   Environment settings

`Secret` is intended for sensitive data such as:

-   Passwords
-   Tokens
-   API keys
-   Certificates

Example:

``` yaml
envFrom:
  - configMapRef:
      name: app-config
```

Secret:

``` yaml
envFrom:
  - secretRef:
      name: app-secret
```

Important interview point:

**Kubernetes Secret data is base64 encoded by default, not automatically
encrypted simply because it is a Secret.**

Encryption at rest can be configured for etcd.

------------------------------------------------------------------------

# 6. Deployment and ReplicaSet

## Q12. What is the difference between Deployment and ReplicaSet?

### Answer

A ReplicaSet ensures that the desired number of Pods are running.

A Deployment manages ReplicaSets and provides higher-level application
rollout functionality.

Deployment capabilities include:

-   Rolling updates
-   Rollbacks
-   ReplicaSet management
-   Revision history
-   Scaling

Example:

``` bash
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
```

------------------------------------------------------------------------

## Q13. How does a Deployment perform a rolling update?

### Answer

When the Pod template changes, the Deployment creates a new ReplicaSet.

The Deployment gradually:

-   Scales up the new ReplicaSet
-   Scales down the old ReplicaSet

The behavior is controlled by:

``` yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
```

This allows controlled replacement of old Pods.

------------------------------------------------------------------------

## Q14. A new deployment version is broken. How do you rollback?

### Answer

Check rollout history:

``` bash
kubectl rollout history deployment/<deployment>
```

Rollback:

``` bash
kubectl rollout undo deployment/<deployment>
```

Then verify:

``` bash
kubectl rollout status deployment/<deployment>
kubectl get pods
```

For a specific revision:

``` bash
kubectl rollout undo deployment/<deployment> --to-revision=<revision>
```

------------------------------------------------------------------------

# 7. Readiness, Liveness and Startup Probes

## Q15. Explain readiness, liveness and startup probes.

### Answer

### Readiness Probe

Answers:

> Can this Pod receive traffic?

If readiness fails, the Pod is removed from Service endpoints.

### Liveness Probe

Answers:

> Is the application still alive?

If liveness repeatedly fails, kubelet restarts the container.

### Startup Probe

Used for slow-starting applications.

While startup probe is running, Kubernetes does not run
liveness/readiness probes in the normal way.

Example:

``` yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
```

------------------------------------------------------------------------

## Q16. What happens if the readiness probe fails?

### Answer

The container normally continues running.

But the Pod becomes **NotReady**, and it is removed from the endpoints
used by the Service.

Therefore:

``` text
Pod Running
      |
Readiness fails
      |
Pod NotReady
      |
Removed from Service endpoints
```

This is different from a liveness failure, which can cause the container
to restart.

------------------------------------------------------------------------

# 8. Scheduling

## Q17. What factors does the scheduler consider when scheduling a Pod?

### Answer

The scheduler considers constraints and available resources such as:

-   CPU
-   Memory
-   Taints/tolerations
-   NodeSelector
-   Node affinity
-   Pod affinity
-   Pod anti-affinity
-   Topology constraints
-   Resource requests
-   Node conditions
-   Scheduling policies/plugins

The scheduler first filters unsuitable nodes and then scores feasible
nodes.

------------------------------------------------------------------------

## Q18. Explain taints and tolerations.

### Answer

A **taint is applied to a node**.

A **toleration is applied to a Pod**.

Example:

``` bash
kubectl taint nodes node1 dedicated=database:NoSchedule
```

Only Pods with a matching toleration can normally be scheduled there.

Example:

``` yaml
tolerations:
  - key: dedicated
    operator: Equal
    value: database
    effect: NoSchedule
```

Important:

**A toleration allows a Pod to tolerate a taint; it does not by itself
force the Pod onto that node.**

------------------------------------------------------------------------

## Q19. What is the difference between nodeSelector and nodeAffinity?

### Answer

`nodeSelector` provides simple label-based node selection.

Example:

``` yaml
nodeSelector:
  disktype: ssd
```

Node affinity provides more expressive rules.

Example:

``` yaml
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

------------------------------------------------------------------------

# 9. Resource Management

## Q20. What is the difference between requests and limits?

### Answer

**Request** is the amount of resource Kubernetes uses for scheduling
decisions.

**Limit** is the maximum resource the container is allowed to consume
for that resource.

Example:

``` yaml
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

For memory, exceeding the limit can result in an OOM kill.

------------------------------------------------------------------------

## Q21. A Pod is repeatedly getting `OOMKilled`. How do you troubleshoot?

### Answer

Run:

``` bash
kubectl describe pod <pod-name>
```

Look for:

``` text
Reason: OOMKilled
Exit Code: 137
```

Then check:

``` bash
kubectl top pod <pod-name>
kubectl top nodes
```

I would compare actual memory usage against the container memory limit.

Then investigate:

-   Memory leak
-   Incorrect application behavior
-   Too-low memory limit
-   Sudden workload increase
-   JVM/Python/Go runtime memory behavior

The correct fix may be increasing the limit, tuning the application, or
fixing a memory leak. Simply increasing the limit is not always the
correct solution.

------------------------------------------------------------------------

# 10. Storage

## Q22. Explain PV, PVC and StorageClass.

### Answer

**PV --- PersistentVolume**

Represents storage available to the cluster.

**PVC --- PersistentVolumeClaim**

A request for storage by a workload.

**StorageClass**

Defines how storage can be dynamically provisioned.

Typical flow:

``` text
Pod
 |
PVC
 |
StorageClass
 |
CSI Driver
 |
Cloud Disk / Storage
 |
PV
```

------------------------------------------------------------------------

## Q23. A Pod is stuck in `Pending` because its PVC is not bound. How do you troubleshoot?

### Answer

Check:

``` bash
kubectl get pvc
kubectl describe pvc <pvc-name>
kubectl get pv
kubectl get storageclass
```

Possible causes:

-   No suitable PV
-   StorageClass missing
-   Incorrect StorageClass
-   CSI driver problem
-   Access mode mismatch
-   Insufficient capacity
-   Provisioner failure

Also check CSI components:

``` bash
kubectl get pods -A | grep -i csi
```

------------------------------------------------------------------------

# 11. RBAC and Security

## Q24. Explain Kubernetes RBAC.

### Answer

RBAC controls **who can perform which actions on which Kubernetes
resources**.

Main objects:

-   Role
-   ClusterRole
-   RoleBinding
-   ClusterRoleBinding

Example:

``` text
User
 |
RoleBinding
 |
Role
 |
Resources + Verbs
```

Typical verbs:

``` text
get
list
watch
create
update
patch
delete
```

A Role is namespace-scoped.

A ClusterRole can provide cluster-wide permissions or reusable
permissions.

------------------------------------------------------------------------

## Q25. A developer says they cannot delete Pods. How do you troubleshoot?

### Answer

First identify the user's identity:

``` bash
kubectl auth whoami
```

Then test authorization:

``` bash
kubectl auth can-i delete pods -n <namespace>
```

If required:

``` bash
kubectl auth can-i delete pods \
  --as=<user> \
  -n <namespace>
```

Then inspect:

``` bash
kubectl get role -n <namespace>
kubectl get rolebinding -n <namespace>
kubectl describe rolebinding <name> -n <namespace>
```

For cluster-wide access:

``` bash
kubectl get clusterrole
kubectl get clusterrolebinding
```

------------------------------------------------------------------------

# 12. NetworkPolicy

## Q26. What is a NetworkPolicy?

### Answer

NetworkPolicy controls network traffic to/from Pods when the cluster's
network plugin supports NetworkPolicy enforcement.

It can restrict:

-   Ingress
-   Egress

Example concept:

``` text
Frontend Pod
     |
     | allowed
     v
Backend Pod

Other Pod
     |
     X denied
     v
Backend Pod
```

A key interview point:

**Creating a NetworkPolicy does not automatically mean every network
plugin will enforce it. The CNI must support NetworkPolicy.**

------------------------------------------------------------------------

# 13. Node Troubleshooting

## Q27. A Kubernetes node becomes `NotReady`. How do you troubleshoot it?

### Answer

Start with:

``` bash
kubectl get nodes
kubectl describe node <node-name>
```

Check conditions:

``` text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

Then investigate the node.

On the node:

``` bash
systemctl status kubelet
journalctl -u kubelet
```

Check container runtime:

``` bash
systemctl status containerd
journalctl -u containerd
```

Also check:

``` bash
df -h
free -m
uptime
```

Possible causes:

-   kubelet failure
-   Container runtime failure
-   Disk full
-   Memory pressure
-   Network problems
-   CNI failure
-   Certificate issues
-   Node resource exhaustion

------------------------------------------------------------------------

# 14. Kubelet

## Q28. What happens if kubelet stops running?

### Answer

The kubelet is responsible for managing Pods on its node and reporting
node status to the API server.

If kubelet stops:

-   Existing containers may continue running temporarily because the
    runtime manages them.
-   Kubernetes loses normal kubelet health/status reporting.
-   The node eventually becomes `NotReady`.
-   Kubernetes may eventually evict workloads depending on node failure
    behavior and configured tolerations/timeouts.

Troubleshooting:

``` bash
systemctl status kubelet
journalctl -u kubelet
```

------------------------------------------------------------------------

# 15. Control Plane Troubleshooting

## Q29. kube-apiserver is unavailable. What is the impact?

### Answer

The API server is the central API endpoint.

If it is unavailable:

-   `kubectl` commands generally fail.
-   Controllers cannot normally communicate with the API server.
-   Scheduler cannot obtain normal API state.
-   New scheduling and control operations are affected.

Existing containers may continue running because the container runtime
and kubelet operate locally, but cluster control functionality is
impaired.

For a highly available cluster, multiple control-plane/API server
instances reduce the impact of a single failure.

------------------------------------------------------------------------

## Q30. What happens if etcd goes down?

### Answer

`etcd` stores Kubernetes control-plane state.

If etcd becomes unavailable:

-   API server cannot reliably read/write cluster state.
-   New changes cannot be persisted.
-   Controllers and scheduler cannot operate normally.
-   Existing workloads may continue running on nodes for some time.

In production, etcd requires:

-   High availability
-   Regular backups
-   Monitoring
-   Careful disaster recovery procedures

------------------------------------------------------------------------

# 16. Logs and Events

## Q31. How do you investigate a failing application Pod?

### Answer

I normally follow this order:

``` bash
kubectl get pod <pod>
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by=.lastTimestamp
```

Then:

``` bash
kubectl exec -it <pod> -- sh
```

if the container is available.

I check:

1.  Pod status
2.  Events
3.  Container state
4.  Exit code
5.  Application logs
6.  Previous container logs
7.  Probes
8.  Resource usage
9.  ConfigMap/Secret
10. Network/dependencies

------------------------------------------------------------------------

# 17. Init Containers

## Q32. What is an init container?

### Answer

An init container runs before the main application containers.

It is useful for initialization tasks such as:

-   Waiting for dependencies
-   Preparing files
-   Database initialization
-   Configuration generation
-   Permission setup

Example:

``` text
Pod
 |
 +-- Init Container
 |
 +-- Main Container
```

All init containers must complete successfully before the application
containers start.

------------------------------------------------------------------------

# 18. Sidecar Containers

## Q33. What is a sidecar container?

### Answer

A sidecar is an additional container running in the same Pod as the main
application.

Containers in the same Pod share:

-   Network namespace
-   Pod IP
-   Volumes mounted into the Pod

Common uses:

-   Log shipping
-   Proxy
-   Service mesh
-   Configuration synchronization
-   Metrics/helper processes

------------------------------------------------------------------------

# 19. StatefulSet vs Deployment

## Q34. When would you use StatefulSet instead of Deployment?

### Answer

Deployment is generally used for stateless applications.

StatefulSet is designed for workloads that need stable identity or
persistent storage characteristics.

StatefulSet provides:

-   Stable Pod names
-   Stable network identity
-   Ordered deployment/scaling behavior
-   Persistent volume association

Examples:

-   Databases
-   Kafka
-   ZooKeeper-like systems
-   Stateful distributed applications

Example Pod names:

``` text
db-0
db-1
db-2
```

------------------------------------------------------------------------

# 20. DaemonSet

## Q35. What is a DaemonSet?

### Answer

A DaemonSet ensures that a Pod runs on eligible nodes.

Common uses:

-   Node monitoring agents
-   Log collectors
-   Security agents
-   CNI components

Example:

``` text
Node 1 -> agent
Node 2 -> agent
Node 3 -> agent
Node 4 -> agent
```

When a new eligible node joins, the DaemonSet controller schedules the
Pod there.

------------------------------------------------------------------------

# 21. Job and CronJob

## Q36. What is the difference between Job and CronJob?

### Answer

A **Job** runs a task until successful completion.

A **CronJob** creates Jobs according to a schedule.

Example:

``` text
CronJob
   |
   +-- Job
        |
        +-- Pod
```

Useful for:

-   Database backups
-   Batch processing
-   Cleanup jobs
-   Scheduled reports

------------------------------------------------------------------------

# 22. HPA

## Q37. Explain Kubernetes Horizontal Pod Autoscaler.

### Answer

HPA automatically changes the number of Pod replicas based on metrics.

For example:

``` text
CPU increases
     |
     v
HPA
     |
     v
Replicas increase
```

Check:

``` bash
kubectl get hpa
kubectl describe hpa <name>
```

HPA commonly uses CPU/memory metrics and can also use custom/external
metrics depending on the metrics stack.

------------------------------------------------------------------------

# 23. HPA vs Cluster Autoscaler

## Q38. What is the difference between HPA and Cluster Autoscaler?

### Answer

**HPA scales Pods.**

``` text
Pod replicas:
3 -> 6
```

**Cluster Autoscaler scales nodes.**

``` text
Worker nodes:
5 -> 7
```

Typical relationship:

``` text
Traffic increases
      |
      v
HPA increases Pods
      |
      v
No available node capacity
      |
      v
Cluster Autoscaler adds nodes
      |
      v
Pending Pods get scheduled
```

------------------------------------------------------------------------

# 24. Production Scenario

## Q39. Application latency suddenly increases in production. How do you troubleshoot?

### Answer

I would not immediately restart Pods.

First establish whether the problem is:

-   Application
-   Pod
-   Node
-   Service
-   Network
-   Database
-   External dependency
-   Infrastructure capacity

Start:

``` bash
kubectl get pods -o wide
kubectl top pods
kubectl top nodes
kubectl get events --sort-by=.lastTimestamp
```

Check application logs:

``` bash
kubectl logs <pod>
```

Check:

``` bash
kubectl describe pod <pod>
kubectl describe svc <service>
kubectl get endpointslices
```

Then correlate with monitoring:

-   CPU
-   Memory
-   Request rate
-   Error rate
-   Latency
-   Database latency
-   Network errors

A senior engineer should identify the bottleneck before applying a
remediation.

------------------------------------------------------------------------

# 25. Deployment Has Zero Available Pods

## Q40. Deployment shows desired replicas but available replicas are zero. What do you check?

### Answer

Start:

``` bash
kubectl get deployment
kubectl describe deployment <name>
kubectl get replicasets
kubectl get pods
```

Then:

``` bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events --sort-by=.lastTimestamp
```

Possible causes:

-   ImagePullBackOff
-   CrashLoopBackOff
-   Failed readiness probe
-   Insufficient resources
-   Scheduling failure
-   Configuration error
-   Secret/ConfigMap missing
-   Application startup failure

------------------------------------------------------------------------

# 26. ImagePullBackOff

## Q41. A Pod is in `ImagePullBackOff`. How do you troubleshoot?

### Answer

Run:

``` bash
kubectl describe pod <pod-name>
```

Look at Events.

Common causes:

-   Incorrect image name
-   Incorrect tag
-   Image does not exist
-   Registry unavailable
-   Private registry authentication failure
-   ImagePullSecret missing
-   Network/DNS problem

Check:

``` bash
kubectl get pod <pod> -o yaml
kubectl get secrets
```

For private registries, verify the Pod's `imagePullSecrets`.

------------------------------------------------------------------------

# 27. Service Types Interview Scenario

## Q42. Explain the difference between ClusterIP, NodePort and LoadBalancer with traffic flow.

### Answer

### ClusterIP

``` text
Pod A
 |
ClusterIP
 |
Pod B
```

Internal cluster access.

### NodePort

``` text
External Client
 |
NodeIP:NodePort
 |
Service
 |
Pod
```

### LoadBalancer

``` text
Internet
 |
Cloud Load Balancer
 |
Service
 |
Pod
```

ClusterIP is normally the internal base abstraction. NodePort and
LoadBalancer provide external exposure mechanisms.

------------------------------------------------------------------------

# 28. kubectl Commands You Should Know

## Q43. Give important Kubernetes troubleshooting commands.

### Answer

### Pods

``` bash
kubectl get pods -A
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl exec -it <pod> -- sh
```

### Nodes

``` bash
kubectl get nodes
kubectl describe node <node>
kubectl top nodes
```

### Services

``` bash
kubectl get svc
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl get endpointslices
```

### Deployments

``` bash
kubectl get deployment
kubectl describe deployment <deployment>
kubectl rollout status deployment/<deployment>
kubectl rollout history deployment/<deployment>
kubectl rollout undo deployment/<deployment>
```

### Events

``` bash
kubectl get events --sort-by=.lastTimestamp
```

### Configuration

``` bash
kubectl get configmap
kubectl get secret
```

### RBAC

``` bash
kubectl auth can-i get pods
kubectl auth can-i delete pods -n <namespace>
kubectl auth whoami
```

------------------------------------------------------------------------

# 29. Senior L2 Scenario Questions

These are particularly important for an interview.

## Q44. Pods are running, Service exists, but users get connection timeout. What complete flow do you inspect?

### Answer

I trace the traffic hop by hop:

``` text
Client
 |
DNS
 |
Load Balancer
 |
Ingress
 |
Service
 |
EndpointSlice
 |
Pod IP
 |
Container Port
 |
Application
```

At each layer I verify:

``` bash
kubectl get ingress
kubectl describe ingress <name>

kubectl get svc
kubectl describe svc <name>

kubectl get endpoints <service>
kubectl get endpointslices

kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
```

I also verify:

-   Security groups/firewalls where applicable
-   NetworkPolicy
-   CNI health
-   DNS
-   Service `port` vs `targetPort`
-   Application listening address
-   Application listening port

------------------------------------------------------------------------

## Q45. A Pod is healthy but traffic is not reaching it. What could be wrong?

### Answer

Possible causes include:

-   Service selector mismatch
-   Pod not Ready
-   Incorrect Service `targetPort`
-   Incorrect Service port
-   EndpointSlice missing the Pod
-   NetworkPolicy blocking traffic
-   Ingress configuration issue
-   Load balancer configuration
-   Application listening on the wrong interface/port
-   DNS issue

I would start with:

``` bash
kubectl get svc
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl get endpointslices
kubectl get pods --show-labels
```

------------------------------------------------------------------------

# 30. Advanced Troubleshooting Scenario

## Q46. A Pod works when accessed directly but fails through the Service. What would you investigate?

### Answer

I would compare:

``` text
Pod IP:Port
```

against:

``` text
Service IP:Port
```

Check:

``` bash
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl get endpointslices
```

I would verify:

-   Service selector
-   Service port
-   targetPort
-   Protocol
-   Endpoint IP/port
-   Readiness
-   NetworkPolicy
-   kube-proxy/CNI behavior

A very common problem is:

``` yaml
ports:
  - port: 80
    targetPort: 8080
```

while the application is actually listening on another port.

------------------------------------------------------------------------

# 31. Production Best Practices

## Q47. What Kubernetes practices would you follow in production?

### Answer

I would emphasize:

1.  Define CPU/memory requests and limits appropriately.
2.  Configure readiness, liveness and startup probes.
3.  Use PodDisruptionBudgets for critical workloads.
4.  Use multiple replicas for highly available applications.
5.  Apply RBAC with least privilege.
6.  Use Secrets appropriately and enable encryption at rest where
    required.
7.  Use NetworkPolicies where appropriate.
8.  Use namespaces for logical isolation.
9.  Use resource quotas and LimitRanges.
10. Monitor cluster and application metrics.
11. Centralize logs.
12. Maintain etcd/control-plane backup and disaster recovery procedures.
13. Use rolling deployments and controlled rollbacks.
14. Pin or otherwise control container image versions rather than
    relying blindly on `latest`.
15. Test Kubernetes manifests in CI before deployment.

------------------------------------------------------------------------

# 32. High-Value Interview Questions --- Rapid Fire

Prepare short answers for these:

### Q48. What is a namespace?

A logical isolation boundary for Kubernetes resources within a cluster.

### Q49. What is kube-proxy?

A node-level component involved in implementing Service networking
rules, traditionally using mechanisms such as iptables or IPVS depending
on configuration.

### Q50. What is CNI?

Container Network Interface. It provides the networking integration used
to configure Pod networking.

### Q51. What is CSI?

Container Storage Interface. It standardizes integration between
Kubernetes and storage systems.

### Q52. What is CRI?

Container Runtime Interface. It defines how kubelet communicates with a
container runtime.

### Q53. What is a finalizer?

A mechanism that prevents an object from being fully deleted until
required cleanup logic is completed.

### Q54. What is an admission controller?

A component/plugin that can validate or mutate API requests after
authentication/authorization and before persistence.

### Q55. What is a PodDisruptionBudget?

It limits voluntary disruptions to maintain application availability.

### Q56. What is a ServiceAccount?

An identity used by workloads running inside Kubernetes for API
authentication and authorization.

### Q57. What is a Custom Resource?

An extension of the Kubernetes API that allows additional resource types
beyond built-in Kubernetes objects.

### Q58. What is an Operator?

A Kubernetes controller pattern that uses APIs and reconciliation logic
to automate management of an application or system.

------------------------------------------------------------------------

# 33. Must-Know Troubleshooting Decision Tree

## Pod Pending

``` text
Pod Pending
    |
    +--> kubectl describe pod
             |
             +--> Insufficient CPU/Memory
             |
             +--> Taint/Toleration
             |
             +--> Affinity
             |
             +--> NodeSelector
             |
             +--> PVC
             |
             +--> ResourceQuota
             |
             +--> Other scheduler event
```

## CrashLoopBackOff

``` text
CrashLoopBackOff
      |
      +--> kubectl logs
      |
      +--> kubectl logs --previous
      |
      +--> kubectl describe pod
      |
      +--> Exit Code?
      |
      +--> OOMKilled?
      |
      +--> Probe failure?
      |
      +--> Configuration?
      |
      +--> Dependency?
```

## Service Not Working

``` text
Service problem
      |
      +--> Service exists?
      |
      +--> Selector correct?
      |
      +--> Endpoints exist?
      |
      +--> EndpointSlice correct?
      |
      +--> targetPort correct?
      |
      +--> Pod Ready?
      |
      +--> NetworkPolicy?
      |
      +--> Ingress/LB?
      |
      +--> DNS?
```

------------------------------------------------------------------------

# 34. Senior L2 Interview Answer Pattern

When the interviewer gives you a production problem, avoid immediately
saying:

> "I will restart the Pod."

Instead answer like this:

``` text
First, I will identify the exact symptom.

Then I will check the Kubernetes object status
and describe it to inspect events.

Next, I will check application logs and previous
container logs if the container has restarted.

After that I will verify resources, probes,
configuration, networking and dependencies.

Once I identify the root cause, I will apply the
least disruptive fix and verify recovery.

Finally, I will add preventive monitoring or
configuration changes so that the issue does not
recur.
```

This demonstrates **L2 troubleshooting maturity**, rather than only
knowledge of `kubectl` commands.

------------------------------------------------------------------------

# 35. Top 15 Questions to Memorize Before the Interview

1.  Pod stuck in Pending --- how do you troubleshoot?
2.  Pod in CrashLoopBackOff --- what do you check?
3.  Pod Running but application inaccessible --- troubleshooting steps?
4.  Service has no endpoints --- why?
5.  Explain ClusterIP, NodePort and LoadBalancer.
6.  Explain traffic flow from user to Pod.
7.  Explain readiness vs liveness vs startup probes.
8.  Explain requests vs limits.
9.  What causes OOMKilled?
10. Node is NotReady --- troubleshooting steps?
11. Explain PV, PVC and StorageClass.
12. Explain taints and tolerations.
13. Explain Deployment, ReplicaSet and rolling update.
14. Explain Kubernetes RBAC and `kubectl auth can-i`.
15. Service works internally but not externally --- how do you
    troubleshoot?

------------------------------------------------------------------------

# Final Interview Tip

For **Senior L2**, interviewers usually care less about whether you can
recite definitions and more about whether you can **systematically
isolate a production problem**.

For every scenario, remember:

``` text
1. Observe
2. Describe
3. Check Events
4. Check Logs
5. Check Dependencies
6. Check Resources
7. Check Networking
8. Identify Root Cause
9. Fix
10. Verify
11. Prevent recurrence
```

That is the mindset you should demonstrate during the interview.
