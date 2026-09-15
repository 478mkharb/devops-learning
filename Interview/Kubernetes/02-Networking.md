# Kubernetes Networking — Interview Questions and Answers

## 1. What is Kubernetes networking?

**Answer:**

Kubernetes networking provides communication between:

- Containers inside the same Pod
- Pods on the same node
- Pods on different nodes
- Pods and Services
- Pods and external systems
- Users and applications through Ingress or LoadBalancer Services

Kubernetes follows four important networking requirements:

1. Every Pod gets its own IP address.
2. Pods can communicate with other Pods without requiring Network Address Translation in the normal Pod network model.
3. Nodes can communicate with all Pods without requiring NAT in the normal Pod network model.
4. The IP address that a Pod sees for itself should be the same IP address that other Pods use to reach it.

The actual implementation is provided by a networking solution, commonly through a CNI plugin.

---

## 2. How do containers inside the same Pod communicate?

**Answer:**

Containers inside the same Pod share the same network namespace. Therefore, they share:

- The same network interface
- The same Pod IP address
- The same routing table
- The same network namespace

They can communicate using `localhost`.

For example, if one container listens on port `8080`, another container in the same Pod can access it using:

```text
http://localhost:8080
```

Example:

```text
Pod
├── Application container: localhost:8080
└── Sidecar container:     localhost:15000
```

The containers do not need to communicate through the Pod IP when they are in the same Pod.

**Important:** Containers in the same Pod must avoid binding to the same IP address and port combination, because they share the same network namespace.

---

## 3. How does Pod-to-Pod communication work when both Pods are on the same node?

**Answer:**

When two Pods run on the same node, the CNI plugin configures network connectivity for both Pods.

In a common bridge-based implementation, the packet flow is:

```text
Frontend Pod eth0
        ↓
Pod-side veth interface
        ↓
Node-side veth interface
        ↓
Linux bridge such as cni0
        ↓
Node-side veth interface of Backend Pod
        ↓
Backend Pod-side veth interface
        ↓
Backend Pod eth0
```

Example:

```text
Frontend Pod: 10.244.1.10
Backend Pod:  10.244.1.20
```

The Frontend Pod can connect directly to:

```text
10.244.1.20:8080
```

The traffic normally stays inside the node. It does not need to travel through the external network.

**Important:** `cni0` is common in bridge-based CNI implementations, but it is not mandatory for every CNI. Some CNIs use routing, eBPF, ENIs, macvlan, ipvlan, or other datapaths.

---

## 4. What is a veth pair and why is it used in Kubernetes networking?

**Answer:**

A veth pair is a pair of connected virtual Ethernet interfaces. Packets entering one end appear at the other end.

For a Pod network, one interface is usually placed inside the Pod network namespace and named `eth0`. The other interface remains in the node's network namespace.

Example:

```text
Pod namespace                 Node namespace
---------------               -------------
eth0  <====================>  vethXXXX
```

The veth pair connects the isolated Pod network namespace to the node networking stack.

A CNI plugin may use veth pairs together with:

- Linux bridges
- Routing tables
- iptables rules
- eBPF programs
- Other networking mechanisms

The exact implementation depends on the CNI plugin.

---

## 5. How does Pod-to-Pod communication work when Pods are on different nodes?

**Answer:**

When the source and destination Pods are on different nodes, the traffic must travel through the node-to-node network.

General flow:

```text
Source Pod
    ↓
Source node CNI datapath
    ↓
Source node routing or encapsulation
    ↓
Underlying node network
    ↓
Destination node routing or decapsulation
    ↓
Destination node CNI datapath
    ↓
Destination Pod
```

Example:

```text
Frontend Pod on Node A: 10.244.1.10
Backend Pod on Node B:  10.244.2.20
```

The CNI plugin may implement this using:

- Overlay networking such as VXLAN
- IP-in-IP encapsulation
- Native routing
- Cloud provider network interfaces
- ENI-based networking
- eBPF-based forwarding

The exact path depends on the networking plugin.

**Interview-ready answer:**

> For different-node communication, the source Pod sends the packet to its node. The CNI forwards it through the node-to-node network using routing, encapsulation, or cloud-native networking. The destination node then delivers the packet to the destination Pod.

---

## 6. Does Kubernetes always use VXLAN for Pod networking?

**Answer:**

No. Kubernetes does not mandate VXLAN.

VXLAN is one possible implementation used by some overlay networking solutions. Other implementations include:

- IP-in-IP
- Native Layer 3 routing
- AWS ENI networking
- Azure VNet integration
- GCP VPC-native networking
- eBPF-based datapaths
- macvlan or ipvlan

The CNI plugin determines how Pod traffic is transported between nodes.

A correct interview statement is:

> VXLAN is a possible CNI implementation for cross-node Pod networking, but it is not a universal Kubernetes requirement.

---

## 7. What is CNI in Kubernetes?

**Answer:**

CNI stands for **Container Network Interface**.

CNI is a specification and plugin model used to configure networking for containers and Pods.

A CNI plugin commonly performs tasks such as:

- Creating the Pod network interface
- Connecting the Pod to the node network
- Assigning an IP address, directly or through an IPAM plugin
- Configuring routes
- Configuring networking rules or datapaths
- Enabling connectivity between Pods and networks
- Cleaning up networking when the Pod is deleted

Examples of Kubernetes networking solutions include:

- Cilium
- Calico
- Flannel
- Weave Net
- Cloud-provider CNI plugins

CNI itself is not one single networking technology. It is the interface through which networking plugins are invoked.

---

## 8. Who invokes the CNI plugin?

**Answer:**

The usual sequence is:

```text
Kubelet
   ↓
Container Runtime
   ↓
CNI Plugin
   ↓
Pod network interface, IP and routes
```

Detailed flow:

1. The scheduler assigns a Pod to a node.
2. The kubelet on that node asks the container runtime to create the Pod sandbox.
3. The container runtime invokes the CNI plugin using the CNI operation, commonly `ADD`.
4. The CNI plugin creates or configures the Pod network interface.
5. An IPAM plugin may allocate the Pod IP address.
6. The CNI plugin configures routes and other networking requirements.
7. When the Pod is deleted, the runtime invokes the CNI `DEL` operation for cleanup.

**Important:** The kubelet normally does not directly create the Pod's network interface. The container runtime invokes the CNI plugin.

---

## 9. What is IPAM in Kubernetes networking?

**Answer:**

IPAM stands for **IP Address Management**.

An IPAM plugin allocates and manages IP addresses for Pods or containers.

Its responsibilities may include:

- Allocating an unused IP address
- Maintaining IP allocation information
- Returning the IP address when a Pod is deleted
- Providing gateway and route information

A common flow is:

```text
Container Runtime
       ↓
CNI Network Plugin
       ↓
IPAM Plugin
       ↓
Pod IP allocation
```

The CNI plugin configures the network, while the IPAM plugin commonly handles IP allocation.

---

## 10. What is the difference between CNI and a CNI plugin?

**Answer:**

**CNI** is the specification and invocation model.

A **CNI plugin** is the actual implementation that configures networking.

Example:

```text
CNI = standard/interface/model
Cilium = networking implementation using CNI
Calico = networking implementation using CNI
Flannel = networking implementation using CNI
```

A CNI plugin may use different technologies internally, such as:

- Linux bridges
- Routing
- VXLAN
- IP-in-IP
- ENIs
- eBPF

Therefore, CNI and a specific CNI plugin should not be treated as exactly the same thing.

---

## 11. What is a Kubernetes Service?

**Answer:**

A Service is a stable logical networking abstraction that exposes a group of Pods.

Pods are temporary and their IP addresses can change when they are recreated. A Service provides a stable access point for those Pods.

A Service normally provides:

- A stable virtual IP called a ClusterIP
- A stable DNS name
- A logical port
- Selection of backend Pods using labels
- Load distribution across eligible endpoints

Example:

```text
Service name: backend-service
ClusterIP:    10.96.0.10
Port:         8080
```

Clients can connect to:

```text
backend-service:8080
```

The Service then forwards traffic to one of the matching backend Pods.

**Important:** A Service is not a Pod, not a network namespace, and not normally a physical network interface.

---

## 12. How does Pod-to-Service communication work?

**Answer:**

When a client Pod sends traffic to a Service, it sends traffic to the Service's virtual ClusterIP or Service DNS name.

Example:

```text
Client Pod:       10.244.1.10
Service ClusterIP: 10.96.0.10:8080
Backend Pod:      10.244.2.20:8080
```

General flow:

```text
Client Pod
    ↓
Service ClusterIP
    ↓
Service dataplane
    ↓
Selected backend endpoint
    ↓
Backend Pod
```

In a traditional setup, kube-proxy programs iptables or IPVS rules to perform the forwarding.

In an eBPF-based setup, a CNI such as Cilium may perform Service load balancing using eBPF instead.

The client does not normally need to know the backend Pod IP.

---

## 13. How does a Service select backend Pods?

**Answer:**

A Service normally uses a label selector.

Example Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - port: 8080
      targetPort: 8080
```

Pods with the following label can become endpoints:

```yaml
metadata:
  labels:
    app: backend
```

Kubernetes tracks matching endpoints through EndpointSlices.

If there are multiple eligible Pods, the Service dataplane can forward traffic to one of them.

---

## 14. What are EndpointSlices?

**Answer:**

EndpointSlices store information about the backend endpoints associated with a Service.

They can contain information such as:

- Endpoint IP addresses
- Ports
- Readiness information
- Serving status
- Termination status
- Endpoint zone information

Example:

```text
Service: backend-service

Endpoints:
- 10.244.2.20:8080
- 10.244.3.30:8080
- 10.244.4.40:8080
```

The Service dataplane uses endpoint information to decide where to forward traffic.

EndpointSlices are preferred over the older Endpoints API for scalability and additional endpoint information.

---

## 15. What is the difference between a Service port, targetPort and nodePort?

**Answer:**

### `port`

The port exposed by the Service.

### `targetPort`

The port on the selected backend Pod where traffic should be delivered.

### `nodePort`

The port opened on each eligible node for a NodePort or LoadBalancer Service.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

Meaning:

```text
Service port: 80
Pod port:     8080
Node port:    30080
```

A client inside the cluster can use:

```text
backend-service:80
```

An external client may use:

```text
<NodeIP>:30080
```

---

## 16. What are the different Kubernetes Service types?

**Answer:**

### ClusterIP

The default Service type. It exposes the Service inside the cluster using a virtual IP.

```yaml
spec:
  type: ClusterIP
```

### NodePort

Exposes the Service on a port on each node.

```yaml
spec:
  type: NodePort
```

### LoadBalancer

Requests an external load balancer from the cloud or infrastructure provider.

```yaml
spec:
  type: LoadBalancer
```

### ExternalName

Maps a Service name to an external DNS name using a CNAME-style response. It does not create a normal ClusterIP-based proxying path.

```yaml
spec:
  type: ExternalName
  externalName: example.com
```

---

## 17. What is a ClusterIP?

**Answer:**

A ClusterIP is a virtual IP assigned to a Service for internal cluster access.

It is not normally assigned to a Pod's network interface.

Example:

```text
Service ClusterIP: 10.96.0.10
Backend Pod IP:    10.244.2.20
```

The ClusterIP remains stable even if backend Pods are replaced.

Traffic sent to the ClusterIP is handled by the Service dataplane, such as:

- kube-proxy using iptables
- kube-proxy using IPVS
- eBPF-based Service handling
- Another supported implementation

---

## 18. What is kube-proxy?

**Answer:**

kube-proxy is a Kubernetes node component traditionally responsible for implementing Service networking.

It watches Services and endpoint information and programs networking rules on each node.

Depending on the mode, kube-proxy can use:

- iptables
- IPVS
- A userspace mode in older implementations

Its rules redirect traffic sent to a Service virtual IP and port toward a selected backend endpoint.

Example:

```text
Client Pod
    ↓
Service ClusterIP
    ↓
iptables or IPVS rules created by kube-proxy
    ↓
Backend Pod
```

**Important:** kube-proxy is mainly associated with Service traffic. Direct Pod-to-Pod traffic does not inherently require kube-proxy.

---

## 19. Is kube-proxy required for every Kubernetes cluster?

**Answer:**

No. kube-proxy is traditional, but it is not the only way to implement Service networking.

Some networking solutions can replace kube-proxy with an eBPF-based datapath.

Examples:

```text
Traditional setup:
CNI + kube-proxy

Some eBPF-based setup:
CNI with eBPF Service handling and kube-proxy replacement
```

The exact requirements depend on the Kubernetes distribution and networking solution.

A correct answer is:

> kube-proxy traditionally implements Services, but a CNI such as Cilium can provide a kube-proxy replacement using eBPF.

---

## 20. What is eBPF and how is it used in Kubernetes networking?

**Answer:**

eBPF is a Linux kernel technology that allows verified programs to run at specific kernel hooks.

In Kubernetes networking, eBPF can be used for:

- Pod packet forwarding
- Service load balancing
- NetworkPolicy enforcement
- Connection tracking
- Load balancing
- Network observability
- Replacing kube-proxy

For example, Cilium uses eBPF for several networking and security functions.

**Important:** eBPF is not the same thing as CNI. A CNI plugin may use eBPF internally.

---

## 21. What is the difference between CNI, kube-proxy and eBPF?

**Answer:**

| Component | Main responsibility |
|---|---|
| CNI | Configures Pod networking, interfaces, IPs and routes |
| kube-proxy | Traditionally implements Service forwarding |
| eBPF | Kernel technology used for networking, Services, policy and observability |
| CoreDNS | Resolves Service and Pod DNS names |

A traditional cluster may use:

```text
CNI + kube-proxy + CoreDNS
```

An eBPF-based cluster may use:

```text
CNI with eBPF + CoreDNS
```

kube-proxy may be present or replaced depending on the configuration.

---

## 22. How does DNS work for a Kubernetes Service?

**Answer:**

Kubernetes normally uses CoreDNS for cluster DNS.

When an application accesses a Service by name, the Pod sends a DNS query to the cluster DNS service.

Example:

```text
backend-service.default.svc.cluster.local
```

CoreDNS resolves the Service name to its ClusterIP.

The flow is:

```text
Application Pod
      ↓ DNS query
CoreDNS
      ↓ DNS response
Service ClusterIP
      ↓ application traffic
Service dataplane
      ↓
Backend Pod
```

CoreDNS resolves the name. It does not normally forward the actual application traffic to the backend Pod.

---

## 23. What is the fully qualified DNS name of a Kubernetes Service?

**Answer:**

The usual Service DNS format is:

```text
<service-name>.<namespace>.svc.<cluster-domain>
```

For example:

```text
backend-service.default.svc.cluster.local
```

Breakdown:

```text
backend-service = Service name
default         = Namespace
svc             = Kubernetes Service DNS zone
cluster.local   = Cluster DNS domain
```

A Service in the same namespace can often be accessed using only:

```text
backend-service
```

---

## 24. What is a headless Service?

**Answer:**

A headless Service is a Service configured with:

```yaml
spec:
  clusterIP: None
```

It does not provide a normal virtual ClusterIP.

Instead, DNS can return the individual Pod IP addresses associated with the Service.

Example:

```text
Headless Service
      ↓ DNS
Pod IP 1
Pod IP 2
Pod IP 3
```

Headless Services are commonly used with StatefulSets and distributed systems that need to discover individual Pods.

**Difference:**

| Normal Service | Headless Service |
|---|---|
| Has ClusterIP | No ClusterIP |
| DNS normally returns virtual Service IP | DNS returns endpoint IPs |
| Service dataplane selects backend | Client can connect to individual endpoints |

---

## 25. What is the difference between Pod IP and Service IP?

**Answer:**

### Pod IP

- Assigned to an individual Pod
- Used for direct Pod-to-Pod communication
- Can change when the Pod is recreated
- Belongs to the Pod network

### Service IP

- Assigned to a Service
- Is a stable virtual IP
- Represents a group of backend Pods
- Is handled by the Service dataplane
- Normally does not belong to a Pod network interface

Example:

```text
Pod IP:     10.244.2.20
Service IP: 10.96.0.10
```

The Service IP remains stable while backend Pod IPs may change.

---

## 26. What is NetworkPolicy?

**Answer:**

NetworkPolicy is a Kubernetes API object used to control which network traffic is allowed to or from selected Pods.

A NetworkPolicy can control:

- Ingress traffic
- Egress traffic
- Source Pods
- Destination Pods
- Namespaces
- IP blocks
- Ports and protocols

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

This policy allows TCP traffic on port `8080` to backend Pods from frontend Pods in the same namespace, assuming the networking implementation supports NetworkPolicy.

**Important:** Creating a NetworkPolicy object does not guarantee enforcement unless the CNI plugin supports and implements NetworkPolicy.

---

## 27. What happens when a NetworkPolicy selects a Pod?

**Answer:**

For a selected Pod, the relevant policy type changes the default behavior.

If an ingress policy selects a Pod:

- Ingress traffic becomes restricted.
- Only traffic allowed by the applicable ingress rules is permitted.

If an egress policy selects a Pod:

- Egress traffic becomes restricted.
- Only traffic allowed by the applicable egress rules is permitted.

Policies are generally additive. If multiple policies apply, the allowed traffic is the union of the traffic allowed by those policies.

A NetworkPolicy is not normally a deny rule by itself. The effect depends on the selected Pods and policy types.

---

## 28. Does NetworkPolicy work with every CNI plugin?

**Answer:**

No. NetworkPolicy enforcement requires support from the networking implementation.

Some CNIs support:

- Standard Kubernetes NetworkPolicy
- Additional policy features
- Layer 3 and Layer 4 rules
- Layer 7 rules in some implementations
- Identity-based enforcement
- Encryption and observability

Examples of policy-capable networking solutions include Calico and Cilium.

The correct answer is:

> NetworkPolicy is a Kubernetes API, but enforcement is performed by a compatible network plugin.

---

## 29. What is Ingress in Kubernetes networking?

**Answer:**

Ingress is a Kubernetes API resource used to define HTTP and HTTPS routing from outside the cluster to Services inside the cluster.

Ingress commonly routes traffic based on:

- Hostname
- URL path
- TLS configuration

Example:

```text
https://example.com/api
        ↓
Ingress Controller
        ↓
backend-service
        ↓
Backend Pods
```

An Ingress resource alone does not process traffic. An Ingress Controller must implement it.

Examples of Ingress Controllers include:

- NGINX Ingress Controller
- HAProxy Ingress
- Traefik
- Cloud-provider ingress controllers
- Gateway API implementations

---

## 30. What is the difference between Ingress and a Service?

**Answer:**

| Service | Ingress |
|---|---|
| Provides access to Pods | Provides HTTP/HTTPS routing to Services |
| Can expose internal or external traffic | Commonly handles north-south web traffic |
| Uses ClusterIP, NodePort or LoadBalancer | Usually requires an Ingress Controller |
| Routes to Pods through Service endpoints | Routes to Services based on host/path rules |
| Works at the Service networking layer | Commonly works at the HTTP/HTTPS layer |

Typical flow:

```text
User
  ↓
Load Balancer
  ↓
Ingress Controller
  ↓
Service
  ↓
Backend Pod
```

---

## 31. What is the difference between north-south and east-west traffic?

**Answer:**

### North-south traffic

Traffic entering or leaving the Kubernetes cluster.

Examples:

```text
Internet → Ingress → Service → Pod
Pod → External Database
```

### East-west traffic

Traffic between workloads inside the cluster or internal platform network.

Examples:

```text
Frontend Pod → Backend Pod
Backend Pod → Database Pod
Service → Backend Pod
```

CNI networking primarily handles Pod connectivity, while Services, Ingress, load balancers and network policies may participate depending on the traffic path.

---

## 32. What is the difference between NodePort and LoadBalancer?

**Answer:**

### NodePort

NodePort exposes a Service on a port on each node.

Example:

```text
<NodeIP>:30080
```

Traffic is then forwarded to the Service backend.

### LoadBalancer

LoadBalancer normally provisions or integrates with an external load balancer through the infrastructure provider.

Typical flow:

```text
External Client
      ↓
External Load Balancer
      ↓
NodePort or provider datapath
      ↓
Service
      ↓
Backend Pod
```

A LoadBalancer Service often uses NodePort internally, but the exact implementation depends on the provider and configuration.

---

## 33. What is kube-proxy iptables mode?

**Answer:**

In iptables mode, kube-proxy creates and maintains iptables rules on each node.

These rules:

- Match traffic sent to Service IPs and ports
- Select a backend endpoint
- Perform destination translation or forwarding
- Support connection tracking and return traffic

Conceptual flow:

```text
Client → ClusterIP:Port
          ↓
      iptables rules
          ↓
    Backend Pod IP:Port
```

iptables mode is widely used and does not require a separate userspace proxy process for every connection.

---

## 34. What is kube-proxy IPVS mode?

**Answer:**

IPVS stands for **IP Virtual Server**.

In IPVS mode, kube-proxy programs Linux IPVS virtual services and real-server entries.

IPVS provides kernel-level load balancing and supports scheduling algorithms such as:

- Round robin
- Least connections
- Destination hashing
- Source hashing

Conceptual flow:

```text
Service virtual IP
        ↓
IPVS virtual service
        ↓
Selected real server / endpoint
```

IPVS can be useful for environments with many Services and endpoints, although the best mode depends on the cluster and workload.

---

## 35. Can Pods communicate directly without a Service?

**Answer:**

Yes. Pods can communicate directly using Pod IP addresses if network connectivity and policy allow it.

Example:

```text
Frontend Pod → 10.244.2.20:8080 → Backend Pod
```

However, direct Pod IPs are not stable because Pods may be deleted and recreated with different IP addresses.

A Service is normally preferred for stable application-to-application communication.

---

## 36. Why should applications usually use a Service instead of a Pod IP?

**Answer:**

Pod IPs are temporary and can change when:

- A Pod is recreated
- A Deployment performs a rollout
- A node fails
- A Pod is rescheduled
- A replica is replaced

A Service provides:

- Stable virtual IP
- Stable DNS name
- Backend endpoint discovery
- Load distribution
- Decoupling between clients and Pod lifecycle

Therefore, applications should normally connect to a Service rather than hardcoding Pod IP addresses.

---

## 37. What is the difference between a Service selector and an EndpointSlice?

**Answer:**

A **Service selector** defines which Pods should be associated with the Service.

Example:

```yaml
selector:
  app: backend
```

An **EndpointSlice** contains the actual endpoint information discovered for that Service, such as:

```text
10.244.2.20:8080
10.244.3.30:8080
```

In simple terms:

```text
Selector = selection rule
EndpointSlice = discovered backend endpoint data
```

---

## 38. What happens if a backend Pod is not ready?

**Answer:**

Kubernetes uses readiness information to determine whether a Pod should receive Service traffic.

If a Pod fails its readiness probe:

- It is normally marked not ready.
- It is generally removed from the set of ready Service endpoints.
- New Service traffic should not be sent to it through normal endpoint selection.

The Pod may still be running, but it should not receive normal application traffic until it becomes ready again.

Readiness and liveness have different purposes:

- Readiness controls whether the Pod should receive traffic.
- Liveness determines whether the container should be restarted.

---

## 39. What happens to Service traffic when a Pod is deleted?

**Answer:**

When a Pod is deleted:

1. Kubernetes updates the Pod status and endpoint information.
2. The endpoint is marked terminating or removed from normal ready endpoints.
3. The Service dataplane updates its forwarding information.
4. New traffic is sent to other eligible endpoints.
5. Existing connections may be allowed to finish depending on the application, connection tracking and termination behavior.

Applications should handle graceful shutdown and connection draining correctly.

---

## 40. How can you check Pod networking from the command line?

**Answer:**

Useful commands include:

```bash
kubectl get pods -o wide
```

Shows Pod IPs and the nodes where Pods are running.

```bash
kubectl get nodes -o wide
```

Shows node IP information.

```bash
kubectl get svc
```

Shows Services and ClusterIPs.

```bash
kubectl get endpointslices
```

Shows Service endpoint information.

```bash
kubectl describe svc <service-name>
```

Shows Service selectors, ports and endpoints.

```bash
kubectl exec -it <pod-name> -- ip addr
```

Shows interfaces inside the Pod.

```bash
kubectl exec -it <pod-name> -- ip route
```

Shows routes inside the Pod.

```bash
kubectl exec -it <pod-name> -- nslookup <service-name>
```

Tests DNS resolution.

```bash
kubectl exec -it <pod-name> -- curl http://<service-name>:<port>
```

Tests application connectivity through a Service.

---

## 41. How do you troubleshoot Pod-to-Pod communication failure?

**Answer:**

Use the following sequence:

### Step 1: Check Pod status

```bash
kubectl get pods -o wide
```

Verify that both Pods are running and note their IP addresses and nodes.

### Step 2: Check Pod interfaces

```bash
kubectl exec -it <pod-name> -- ip addr
```

Verify that the Pod has an interface and IP address.

### Step 3: Check routes

```bash
kubectl exec -it <pod-name> -- ip route
```

Verify that routes exist for the Pod network.

### Step 4: Test direct connectivity

```bash
kubectl exec -it <source-pod> -- curl http://<destination-pod-ip>:<port>
```

### Step 5: Check NetworkPolicy

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy-name>
```

A policy may be blocking ingress or egress traffic.

### Step 6: Check CNI components

```bash
kubectl get pods -n kube-system
```

Look for CNI-related Pods and their logs.

### Step 7: Check node routes and interfaces

On the relevant node:

```bash
ip addr
ip route
```

### Step 8: Check firewall and security rules

For cloud environments, check:

- Security groups
- Network ACLs
- Route tables
- Host firewall
- Cloud network routes

---

## 42. How do you troubleshoot Service connectivity failure?

**Answer:**

Use the following process:

### Step 1: Check the Service

```bash
kubectl get svc <service-name>
kubectl describe svc <service-name>
```

Verify:

- ClusterIP
- Service port
- Target port
- Selector

### Step 2: Check EndpointSlices

```bash
kubectl get endpointslices
```

Verify that backend endpoint IPs and ports exist.

### Step 3: Check Pod labels

```bash
kubectl get pods --show-labels
```

Ensure that the Pod labels match the Service selector.

### Step 4: Check readiness

```bash
kubectl get pods
```

A Pod that is running but not ready may not receive Service traffic.

### Step 5: Test DNS

```bash
kubectl exec -it <client-pod> -- nslookup <service-name>
```

### Step 6: Test the Service by ClusterIP

```bash
kubectl exec -it <client-pod> -- curl http://<cluster-ip>:<port>
```

### Step 7: Test the backend Pod directly

```bash
kubectl exec -it <client-pod> -- curl http://<pod-ip>:<target-port>
```

If direct Pod access works but Service access fails, investigate:

- Service selector
- EndpointSlices
- kube-proxy or eBPF Service datapath
- Service port and targetPort
- NetworkPolicy

---

## 43. What is the difference between `port` and `targetPort` in a Service?

**Answer:**

`port` is the port exposed by the Service.

`targetPort` is the port on the backend Pod.

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

The client connects to:

```text
Service:80
```

The Service forwards traffic to:

```text
Pod:8080
```

If `targetPort` is omitted, it commonly defaults to the same value as `port`, unless a named port or other configuration changes the behavior.

---

## 44. What is a named port in Kubernetes networking?

**Answer:**

A named port allows a Service to refer to a Pod port by name rather than a fixed number.

Pod example:

```yaml
ports:
  - name: http
    containerPort: 8080
```

Service example:

```yaml
ports:
  - port: 80
    targetPort: http
```

This allows the Service to forward traffic to the Pod port named `http`.

Named ports can improve readability and reduce errors when port numbers change.

---

## 45. Can a Service route traffic to Pods in another namespace?

**Answer:**

A normal Service selects Pods within its own namespace. A Service selector does not normally select Pods across namespaces.

To communicate with a Service in another namespace, use the Service DNS name:

```text
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
backend-service.production.svc.cluster.local
```

Cross-namespace access may still be restricted by NetworkPolicy.

---

## 46. What is the difference between `localhost`, Pod IP and Service DNS?

**Answer:**

### `localhost`

Used for communication inside the same network namespace. Containers in the same Pod can use it.

### Pod IP

Used to communicate directly with a particular Pod.

### Service DNS

Used to access a stable Service endpoint by name.

Example:

```text
localhost:8080
10.244.2.20:8080
backend-service.default.svc.cluster.local:8080
```

Use:

- `localhost` for containers in the same Pod
- Pod IP for direct troubleshooting or special use cases
- Service DNS for normal application-to-application communication

---

## 47. What is MTU and why is it important in Kubernetes networking?

**Answer:**

MTU stands for **Maximum Transmission Unit**. It is the largest packet size that can be transmitted over a network interface without fragmentation at that layer.

Overlay networks add extra headers for encapsulation. For example, VXLAN adds overhead to the original packet.

If the underlying network supports an MTU of `1500`, an overlay network may need to use a smaller Pod interface MTU to leave room for encapsulation headers.

Incorrect MTU configuration can cause:

- Slow connections
- Failed large requests
- Broken TLS connections
- Fragmentation
- Intermittent application failures

Useful commands:

```bash
ip link
ip addr
```

Check the MTU configured on Pod and node interfaces.

---

## 48. What is NAT and how can it appear in Kubernetes networking?

**Answer:**

NAT stands for **Network Address Translation**. It changes source or destination IP addresses, and sometimes ports, as packets pass through a networking device or rule set.

In Kubernetes, NAT may be used for:

- Service ClusterIP forwarding
- NodePort traffic
- Egress traffic to external networks
- Masquerading Pod IPs when required by the network design
- LoadBalancer or external traffic handling

The Kubernetes Pod networking model aims to allow direct Pod communication without unnecessary NAT, but specific implementations may use NAT for Services and external traffic.

---

## 49. What is source NAT and destination NAT in Kubernetes?

**Answer:**

### Source NAT, or SNAT

Changes the source IP address of a packet.

A common example is Pod traffic going to an external network where the Pod IP is translated to a node or gateway IP.

### Destination NAT, or DNAT

Changes the destination IP address of a packet.

A common example is traffic sent to a Service ClusterIP being redirected to a backend Pod IP.

Conceptually:

```text
Before DNAT:
Destination = Service ClusterIP

After DNAT:
Destination = Backend Pod IP
```

The exact implementation depends on kube-proxy, the CNI, cloud networking and traffic policy settings.

---

## 50. What is `externalTrafficPolicy`?

**Answer:**

`externalTrafficPolicy` controls how externally received traffic is handled by a Service, especially with NodePort and LoadBalancer Services.

### `Cluster`

Traffic can be forwarded to endpoints on any node.

Advantages:

- Better distribution across endpoints
- Can use all available backend Pods

Possible behavior:

- Source IP may be changed through SNAT depending on the path
- Traffic may cross nodes

### `Local`

Traffic is sent only to endpoints local to the receiving node.

Advantages:

- Can preserve the original client source IP
- Avoids an additional cross-node hop in supported paths

Trade-off:

- A node without a local endpoint may not be able to serve traffic in the same way.

Example:

```yaml
spec:
  externalTrafficPolicy: Local
```

---

## 51. What is `internalTrafficPolicy`?

**Answer:**

`internalTrafficPolicy` controls whether internal Service traffic can use endpoints on any node or only endpoints local to the client node.

### `Cluster`

Traffic may use endpoints across the cluster.

### `Local`

Traffic is restricted to endpoints local to the client node.

Example:

```yaml
spec:
  internalTrafficPolicy: Local
```

This can be useful when applications want to reduce cross-node traffic or use node-local endpoints.

---

## 52. What is a Network Namespace in Kubernetes?

**Answer:**

A network namespace is a Linux isolation mechanism that provides an independent network view.

A network namespace can have its own:

- Network interfaces
- IP addresses
- Routing table
- Firewall rules
- Network sockets

A Pod normally has its own network namespace. Containers in that Pod share it.

The node has its own network namespace, which contains node interfaces and the node networking stack.

A veth pair can connect the Pod network namespace to the node network namespace.

---

## 53. What is the role of the Pod `eth0` interface?

**Answer:**

`eth0` is commonly the primary network interface inside a Pod network namespace.

It provides the Pod with:

- Its Pod IP address
- Connectivity to the Pod network
- A default route or other routes
- Access to other Pods and Services, subject to policy

The interface name may vary in special networking configurations, but `eth0` is the common default.

---

## 54. What is the role of the node-side veth interface?

**Answer:**

The node-side veth interface is the host-side partner of the Pod's network interface.

It connects the Pod network namespace to the node network namespace.

The node-side interface may be:

- Attached to a Linux bridge
- Used with routing rules
- Processed by eBPF programs
- Integrated into another CNI datapath

It is normally visible from the node, not from inside the Pod.

---

## 55. How can you identify the node on which a Pod is running?

**Answer:**

Use:

```bash
kubectl get pods -o wide
```

Example output conceptually:

```text
NAME       READY   STATUS    IP            NODE
frontend   1/1     Running   10.244.1.10   node-a
backend    1/1     Running   10.244.2.20   node-b
```

This command helps determine whether communication is:

- Same-node
- Cross-node

That distinction is useful when troubleshooting networking.

---

## 56. How do you verify that a Service has backend endpoints?

**Answer:**

Use:

```bash
kubectl get endpointslices
```

You can also inspect the Service:

```bash
kubectl describe svc <service-name>
```

Check:

- Service selector
- Endpoint IPs
- Endpoint ports
- Ready state
- Whether the Service has no endpoints

If a Service has no endpoints, common causes include:

- Incorrect selector
- Wrong namespace
- Pods not ready
- Pods not running
- Incorrect port configuration

---

## 57. What is the difference between a Service with selectors and a Service without selectors?

**Answer:**

### Service with selector

Kubernetes automatically discovers matching Pods and creates endpoint information.

### Service without selector

Kubernetes does not automatically select Pods based on labels. Endpoint information may be managed separately, for example through manually created EndpointSlices.

A selector-less Service can be used for cases where the backend is outside the normal Pod selection model, but it requires careful endpoint management.

---

## 58. Why can a Pod access another Pod by IP but not by Service name?

**Answer:**

If direct Pod IP access works but Service name access fails, investigate DNS and Service configuration separately.

Possible causes:

- CoreDNS is not running
- DNS configuration inside the Pod is incorrect
- Wrong Service name
- Wrong namespace
- Service has no endpoints
- Service port or targetPort is incorrect
- NetworkPolicy blocks DNS traffic
- The application is using the wrong protocol or port

Useful commands:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
kubectl exec -it <pod-name> -- nslookup <service-name>
```

---

## 59. Why can a Service resolve through DNS but still fail to connect?

**Answer:**

DNS resolution only confirms that the name was resolved. It does not prove that the application path is working.

Possible causes include:

- No ready endpoints
- Wrong Service port
- Wrong targetPort
- Application not listening on the expected port
- NetworkPolicy blocking traffic
- kube-proxy or eBPF Service datapath issue
- CNI connectivity issue
- Backend application failure
- Protocol mismatch, such as HTTP versus HTTPS

Troubleshoot in this order:

```text
DNS resolution
    ↓
Service ClusterIP reachability
    ↓
Endpoint availability
    ↓
Direct Pod IP connectivity
    ↓
Application port and protocol
```

---

## 60. Give a complete interview answer for Kubernetes Pod networking.

**Answer:**

> Every Kubernetes Pod receives its own IP address from the cluster Pod network. Containers inside the same Pod share a network namespace and communicate using localhost.
>
> When two Pods are on the same node, the CNI connects their network namespaces through a local datapath. In a common bridge-based implementation, each Pod has an `eth0` interface connected through a veth pair to the node, and the node-side interfaces may be connected through a Linux bridge such as `cni0`. The traffic normally remains inside the node.
>
> When Pods are on different nodes, traffic goes from the source Pod to the source node, across the node-to-node network, and then to the destination Pod through the destination node's CNI datapath. The CNI may use routing, VXLAN, IP-in-IP, ENIs, native cloud networking or eBPF, depending on the implementation.
>
> For Service communication, the client sends traffic to a stable Service ClusterIP or Service DNS name. CoreDNS resolves the Service name to the ClusterIP. The Service dataplane then selects a backend endpoint using endpoint information. Traditionally kube-proxy implements this using iptables or IPVS, while some CNIs use eBPF and may replace kube-proxy.
>
> NetworkPolicy can restrict ingress and egress traffic if the CNI supports enforcement. Ingress and LoadBalancer mechanisms provide external access to Services, while DNS provides service discovery.

---

## 61. Explain the complete traffic path from a client Pod to a backend Pod through a Service.

**Answer:**

Assume:

```text
Client Pod:       10.244.1.10
Service ClusterIP: 10.96.0.10:8080
Backend Pod:      10.244.2.20:8080
```

The traffic path is:

```text
Client application
        ↓
Service DNS lookup through CoreDNS, if a name is used
        ↓
Service ClusterIP:8080
        ↓
Service dataplane
        ↓
kube-proxy iptables/IPVS or eBPF implementation
        ↓
Selected backend endpoint 10.244.2.20:8080
        ↓
Backend Pod application
```

If the backend Pod is on another node, the packet then travels through the cross-node CNI network.

The complete path may be:

```text
Client Pod
    ↓
Client node CNI
    ↓
Service forwarding rules
    ↓
Node-to-node network
    ↓
Destination node CNI
    ↓
Backend Pod
```

The exact order of some processing steps depends on the CNI, kube-proxy mode, traffic policy and kernel datapath.

---

## 62. What are the most common Kubernetes networking mistakes in interviews?

**Answer:**

Common incorrect statements include:

### Incorrect: Every CNI uses `cni0`

Correct statement:

> `cni0` is common in bridge-based implementations, but not mandatory for every CNI.

### Incorrect: Kubernetes always uses VXLAN

Correct statement:

> VXLAN is one possible cross-node networking implementation.

### Incorrect: A Service is a Pod

Correct statement:

> A Service is a logical virtual abstraction that exposes backend endpoints.

### Incorrect: CoreDNS forwards application traffic

Correct statement:

> CoreDNS resolves names. The Service dataplane handles application traffic.

### Incorrect: kube-proxy handles all Pod-to-Pod traffic

Correct statement:

> kube-proxy traditionally handles Service traffic. Direct Pod-to-Pod traffic is primarily handled by the CNI and node networking.

### Incorrect: eBPF and CNI are the same thing

Correct statement:

> eBPF is a Linux kernel technology that a CNI may use internally.

### Incorrect: NetworkPolicy is enforced automatically by Kubernetes itself

Correct statement:

> NetworkPolicy enforcement is provided by a compatible networking plugin.

---

## 63. What is the best short answer if the interviewer asks, “How do Pods communicate?”

**Answer:**

> Pods communicate using Pod IP addresses provided by the CNI networking layer. If the Pods are on the same node, traffic uses the local CNI datapath, commonly veth interfaces and a bridge or routing mechanism. If they are on different nodes, traffic crosses the node-to-node network using routing, overlay encapsulation, cloud networking or eBPF, depending on the CNI. For stable communication, applications normally use a Service, where the Service ClusterIP is translated or forwarded to a selected backend Pod by kube-proxy or an eBPF Service datapath.
