# Sysctl

---

## 🌐 net.ipv4.ip_forward

```
sysctl -w net.ipv4.ip_forward=1
```

* Read: `sysctl net.ipv4.ip_forward`
* Default: 0 | Min: 0 | Max: 1
* Why: Enables packet forwarding (host → router behavior)
* When to use: Kubernetes nodes, Docker hosts, NAT/VPN; pods/containers can’t reach other networks

---

## 🚀 net.core.somaxconn

```
sysctl -w net.core.somaxconn=1024
```

* Read: `sysctl net.core.somaxconn`
* Default: 128 | Min: 1 | Max: ~65535
* Why: Increases accept queue (connections waiting for app)
* When to use: Nginx/API under burst traffic; 502/504 or connection drops with low CPU

---

## 🚀 net.ipv4.tcp_max_syn_backlog

```
sysctl -w net.ipv4.tcp_max_syn_backlog=4096
```

* Read: `sysctl net.ipv4.tcp_max_syn_backlog`
* Default: 256–1024 | Min: 128 | Max: ~65535
* Why: Increases SYN queue (half-open connections)
* When to use: High SYN_RECV in `ss -s`, traffic spikes, initial connection timeouts

---

## 🔁 net.ipv4.ip_local_port_range

```
sysctl -w net.ipv4.ip_local_port_range="1024 65535"
```

* Read: `sysctl net.ipv4.ip_local_port_range`
* Default: 32768–60999 | Min: 1024 | Max: 65535
* Why: Expands ephemeral ports for outbound connections
* When to use: Microservices/high outbound calls; errors: "Cannot assign requested address"

---

## 🔁 net.ipv4.tcp_tw_reuse

```
sysctl -w net.ipv4.tcp_tw_reuse=1
```

* Read: `sysctl net.ipv4.tcp_tw_reuse`
* Default: 0 | Min: 0 | Max: 1
* Why: Reuses TIME_WAIT sockets for new outbound connections
* When to use: Many short-lived requests; high TIME_WAIT in `ss -s`

---

## 🧠 vm.swappiness

```
sysctl -w vm.swappiness=10
```

* Read: `sysctl vm.swappiness`
* Default: 60 | Min: 0 | Max: 100
* Why: Controls RAM vs swap usage (lower = prefer RAM)
* When to use: Databases/Redis; latency spikes due to swap

---

## 🧠 vm.overcommit_memory

```
sysctl -w vm.overcommit_memory=1
```

* Read: `sysctl vm.overcommit_memory`
* Default: 0 | Min: 0 | Max: 2
* Why: Memory allocation policy (allow allocations beyond RAM)
* When to use: Redis/JVM; avoid allocation failures (Redis recommends 1)

---

## 📂 fs.file-max

```
sysctl -w fs.file-max=2097152
```

* Read: `sysctl fs.file-max`
* Default: very high (system dependent) | Min: 0 | Max: 64-bit limit
* Why: System-wide max file descriptors (connections/files)
* When to use: High connections (Nginx/Kafka/ES); errors: "Too many open files" (also raise `ulimit -n`/`LimitNOFILE`)

---

## 🔧 Apply & Persist (one-liners)

```
sysctl --system
```

* Loads all configs from `/etc/sysctl.conf` and `/etc/sysctl.d/*.conf`
* Persist by adding lines to `/etc/sysctl.d/99-custom.conf`
