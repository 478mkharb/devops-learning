# Types of Docker Networks

This document explains the **types of networks in Docker**, their purpose, use cases, and interview-relevant points.

---

## 1. Bridge Network (Default)

### What it is

* Default network type in Docker
* Containers get private IPs
* Uses NAT for external access

### Key Points

* Container-to-container communication on same host
* Port mapping required to access from host

### Example

```bash
docker run -d --name web nginx
```

### Use Case

* Local development
* Single-host container communication

---

## 2. Host Network

### What it is

* Container shares host’s network stack
* No isolation between container and host network

### Key Points

* No port mapping required
* Faster networking
* Port conflicts possible

### Example

```bash
docker run --network host nginx
```

### Use Case

* High-performance networking
* Monitoring agents

---

## 3. None Network

### What it is

* Completely disables networking for container

### Key Points

* No IP address
* No external or internal connectivity

### Example

```bash
docker run --network none nginx
```

### Use Case

* Security-sensitive workloads
* Batch jobs without networking

---

## 4. Overlay Network

### What it is

* Multi-host networking
* Used in Docker Swarm

### Key Points

* Enables container communication across nodes
* Uses VXLAN tunneling

### Use Case

* Microservices across multiple Docker hosts

---

## 5. Macvlan Network

### What it is

* Assigns MAC and IP directly from physical network

### Key Points

* Container appears as a physical device
* Bypasses Docker bridge

### Use Case

* Legacy applications
* Network-level visibility required

---

## 6. Custom Bridge Network

### What it is

* User-defined bridge network

### Key Points

* Built-in DNS resolution
* Better isolation than default bridge

### Example

```bash
docker network create mynet
docker run --network mynet nginx
```

---

## Comparison Table

| Network Type  | Scope        | Use Case              |
| ------------- | ------------ | --------------------- |
| Bridge        | Single host  | Default containers    |
| Host          | Single host  | Performance           |
| None          | Single host  | Isolation             |
| Overlay       | Multi-host   | Swarm / microservices |
| Macvlan       | Physical LAN | Legacy apps           |
| Custom Bridge | Single host  | Better isolation      |

---

## Interview One-Liner

> Docker supports bridge, host, none, overlay, and macvlan networks, each designed for different levels of isolation, performance, and multi-host communication.

---

## Best Practice

* Use **custom bridge networks** for applications
* Avoid host network unless required
* Use overlay only for multi-host setups
