# 🌐 What is CNI (Container Network Interface)?

## 📌 Definition

**CNI (Container Network Interface)** is a **specification and set of libraries** that defines **how container runtimes configure networking for containers and pods**.

In Kubernetes, CNI is the **standard way** to:

* Assign IP addresses to Pods
* Set up routing between Pods
* Enable Pod-to-Pod, Pod-to-Service, and external communication

Kubernetes itself **does NOT implement networking**. Instead, it relies on **CNI plugins** to handle networking.

---

## 🧠 Why CNI is Needed

Kubernetes networking has some strict requirements:

* Every Pod gets its **own IP address**
* Pods can communicate **without NAT**
* Nodes can communicate with all Pods
* Services can load-balance traffic

CNI plugins ensure these rules are followed.

---

## 🏗️ How CNI Works (High-Level Flow)

```
Pod Created
   ↓
Kubelet detects Pod
   ↓
Kubelet calls CNI plugin
   ↓
CNI plugin:
  - Allocates IP
  - Creates veth pair
  - Configures routing
   ↓
Pod is network-ready
```

---

## 📂 CNI Components

### 1️⃣ CNI Specification

Defines:

* How container runtimes call plugins
* Input/output formats (JSON)

### 2️⃣ CNI Plugins

Executable binaries responsible for:

* Network creation
* IP allocation
* Routing

### 3️⃣ CNI Configuration Files

Stored at:

```
/etc/cni/net.d/
```

Example config file:

```json
{
  "cniVersion": "0.4.0",
  "name": "mynet",
  "type": "bridge",
  "bridge": "cni0",
  "ipam": {
    "type": "host-local",
    "subnet": "10.244.0.0/16"
  }
}
```

---

## 🔌 Popular CNI Plugins

### 🔹 1. Flannel

**Best for:** Simple setups

**How it works:**

* Uses an **overlay network** (VXLAN)
* Assigns each node a subnet

**Pros:**

* Easy to set up
* Lightweight

**Cons:**

* No NetworkPolicy support

**Use Case:**

* Learning Kubernetes
* Small clusters

---

### 🔹 2. Calico

**Best for:** Production workloads

**How it works:**

* Uses **BGP routing** or VXLAN
* Can work **without overlays**

**Pros:**

* High performance
* Supports NetworkPolicies
* Works well at scale

**Cons:**

* Slightly complex to configure

**Use Case:**

* Secure enterprise clusters

---

### 🔹 3. Weave Net

**Best for:** Easy multi-host networking

**How it works:**

* Creates a mesh overlay network
* Encrypts traffic by default

**Pros:**

* Simple installation
* Built-in encryption

**Cons:**

* Slower than Calico

---

### 🔹 4. Cilium

**Best for:** Advanced networking & security

**How it works:**

* Uses **eBPF** instead of iptables
* Provides L3–L7 visibility

**Pros:**

* Very fast
* Advanced observability
* Fine-grained security

**Cons:**

* Requires newer kernels

---

### 🔹 5. Amazon VPC CNI (AWS)

**Best for:** EKS clusters

**How it works:**

* Pods get **real VPC IPs**
* No overlay network

**Pros:**

* Native AWS networking
* High performance

**Cons:**

* IP exhaustion risk

---

## 📊 CNI Plugin Comparison

| Plugin  | Overlay  | NetworkPolicy | Performance | Complexity |
| ------- | -------- | ------------- | ----------- | ---------- |
| Flannel | ✅        | ❌             | Medium      | Low        |
| Calico  | Optional | ✅             | High        | Medium     |
| Weave   | ✅        | ✅             | Medium      | Low        |
| Cilium  | Optional | ✅             | Very High   | High       |
| AWS VPC | ❌        | Limited       | Very High   | Medium     |

---

## 🧪 Example: Pod Networking with Calico

```
Pod A (10.244.1.2) ─────▶ Pod B (10.244.2.3)
       │                     │
       └── Calico routes via BGP ──┘
```

No NAT, direct routing between Pods.

---

## ⚙️ How to Check CNI in Your Cluster

```bash
ls /etc/cni/net.d/
```

```bash
kubectl get pods -n kube-system
```

Look for:

* calico-node
* flannel
* cilium
* weave-net

---

## 📝 Key Takeaways

* CNI is **mandatory** for Kubernetes networking
* Kubernetes delegates networking to CNI plugins
* Choice of CNI impacts **performance, security, and scalability**
* Calico and Cilium are most common in production

---

If you want, I can also add:

* ✅ CNI interview questions
* ✅ CNI troubleshooting guide
* ✅ YAML-based real-world examples
* ✅ Comparison with Docker networking

Just say the word 🚀
