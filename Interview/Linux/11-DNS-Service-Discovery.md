# Linux for DevOps — Topic 11: DNS and Service Discovery

## Scope

This topic covers DNS and service discovery from a DevOps perspective:

- EC2 private DNS
- Route 53
- NGINX and reverse proxies
- Jenkins, Ansible, and monitoring
- Kubernetes service discovery
- Internal APIs and microservices
- DNS failures in production

> **Core principle:** DNS maps names to endpoints. It does not prove that the endpoint is healthy or reachable.

---

# 1. What is DNS?

DNS translates names into records.

```text
employee-api.internal -> 10.0.2.15
```

Common record types:

| Record | Purpose |
|---|---|
| A | Hostname to IPv4 address |
| AAAA | Hostname to IPv6 address |
| CNAME | Alias to another hostname |
| MX | Mail server |
| NS | Authoritative name server |
| TXT | Text, verification, SPF and other metadata |
| SRV | Service location including port |
| PTR | Reverse lookup from IP to name |

---

# 2. DNS resolution flow

```text
Application
   |
   v
Local resolver configuration
   |
   v
Recursive DNS resolver
   |
   v
Root servers
   |
   v
TLD servers
   |
   v
Authoritative DNS server
   |
   v
DNS response
```

In practice, the recursive resolver often serves a cached response without querying the full hierarchy.

Check:

```bash
cat /etc/resolv.conf
resolvectl status
```

---

# 3. Essential DNS commands

```bash
getent hosts example.com
getent ahostsv4 example.com
resolvectl query example.com
dig example.com
dig A example.com
dig AAAA example.com
dig CNAME app.example.com
dig MX example.com
dig NS example.com
dig TXT example.com
dig +short example.com
```

Query a specific resolver:

```bash
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com
```

Reverse lookup:

```bash
dig -x 10.0.2.15
```

Trace delegation:

```bash
dig +trace example.com
```

---

# 4. `/etc/resolv.conf`

Inspect:

```bash
cat /etc/resolv.conf
```

Typical entries:

```text
nameserver 127.0.0.53
options edns0 trust-ad
search internal.example
```

On systems using `systemd-resolved`, `127.0.0.53` may be a local stub resolver.

Inspect actual resolver state:

```bash
resolvectl status
resolvectl dns
resolvectl domain
```

Do not assume `/etc/resolv.conf` alone explains the full resolver configuration.

---

# 5. `/etc/hosts` vs DNS

Example:

```text
10.0.2.15 employee-api.internal employee-api
```

Check:

```bash
cat /etc/hosts
getent hosts employee-api
cat /etc/nsswitch.conf
```

Use `/etc/hosts` for small, controlled static mappings. Use DNS or service discovery for dynamic infrastructure.

A stale hosts-file entry can cause a server to connect to the wrong endpoint.

---

# 6. DNS caching and TTL

TTL means **Time To Live**. It controls how long a resolver may cache a DNS response.

Example:

```text
app.example.com. 300 IN A 203.0.113.10
```

A TTL of `300` means 300 seconds, subject to resolver behavior and DNS implementation.

Important:

- Changing authoritative DNS does not instantly update every cached resolver.
- Low TTL helps planned changes but increases DNS query volume.
- TTL is not a health check.
- Existing TCP connections are not automatically moved when DNS changes.
- Applications may cache DNS longer than expected.

Check TTL:

```bash
dig app.example.com
```

---

# 7. Authoritative vs recursive DNS

| Authoritative server | Recursive resolver |
|---|---|
| Owns or serves the zone data | Resolves answers for clients |
| Publishes records | Caches answers |
| Responds authoritatively | May return cached data |
| Example: hosted DNS zone | Example: corporate/VPC resolver |

Useful command:

```bash
dig +norecurse example.com
```

The answer depends on whether the queried server is authoritative for the name.

---

# 8. DNS failure vs connectivity failure

Use this sequence:

```bash
getent hosts api.internal
dig api.internal
ip route get 10.0.2.15
nc -vz api.internal 8080
curl -v http://api.internal:8080/health
```

Interpretation:

| Result | Likely problem |
|---|---|
| Name does not resolve | DNS, resolver, zone, or record |
| Name resolves but port fails | Route, firewall, SG, NACL, listener |
| TCP succeeds but TLS fails | Certificate, SNI, protocol |
| HTTP returns 5xx | Proxy or application |
| Old IP returned | Cache, TTL, hosts file, split DNS |

---

# 9. Split-horizon DNS

Split-horizon DNS returns different answers depending on where the query originates.

Example:

```text
Internal clients -> app.example.com -> 10.0.2.15
Internet clients -> app.example.com -> public load balancer
```

Useful for:

- Private EC2 services
- Internal APIs
- Corporate networks
- Hybrid cloud
- Avoiding public exposure of private endpoints

Troubleshoot from the same network location as the application:

```bash
dig app.example.com
```

A DNS answer from your laptop may differ from the answer inside a VPC.

---

# 10. AWS Route 53 concepts

Route 53 commonly provides:

- Public hosted zones
- Private hosted zones
- Health checks
- Routing policies
- Alias records
- Weighted routing
- Failover routing
- Latency-based routing
- Geolocation routing

A private hosted zone must be associated with the relevant VPCs.

Typical private design:

```text
employee-api.internal -> private EC2 IP or internal load balancer
```

Avoid hardcoding changing EC2 private IPs in application configuration when DNS can represent the service.

---

# 11. Route 53 routing policies

| Policy | Use case |
|---|---|
| Simple | Basic DNS answer |
| Weighted | Split traffic between versions/environments |
| Latency-based | Route to lowest-latency region |
| Failover | Primary/secondary design |
| Geolocation | Location-based answers |
| Geoproximity | Geographic routing with bias |
| Multivalue answer | Return multiple healthy values |

DNS routing is not the same as an application load balancer. DNS decisions happen at resolution time and are affected by caching.

---

# 12. DNS and NGINX

Example:

```nginx
upstream employee_api {
    server employee-api.internal:8080;
}

server {
    listen 80;
    location /api/ {
        proxy_pass http://employee_api;
    }
}
```

Validate:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Troubleshoot:

```bash
getent hosts employee-api.internal
curl -v http://employee-api.internal:8080/health
sudo tail -f /var/log/nginx/error.log
```

Possible issue:

- DNS resolves during startup but the backend IP later changes.
- NGINX may not continuously re-resolve a static upstream definition in the way you expect.
- For dynamic endpoints, use an appropriate resolver and configuration pattern.

---

# 13. DNS in Jenkins and Ansible

Jenkins may run on a different host, agent, subnet, or network namespace than your SSH session.

Check from the actual execution environment:

```bash
hostname
cat /etc/resolv.conf
getent hosts employee-api.internal
nc -vz employee-api.internal 8080
```

Ansible connectivity often depends on:

- Controller DNS
- Dynamic inventory
- Private hosted zones
- Bastion/proxy settings
- SSM or SSH transport
- Correct inventory hostnames

Do not validate DNS only from your laptop if Jenkins or Ansible executes elsewhere.

---

# 14. Service discovery

Service discovery lets applications find services without hardcoding every endpoint.

Common approaches:

1. DNS names
2. Load balancer names
3. Cloud service registries
4. Consul
5. Kubernetes Services
6. Internal API gateways
7. Static inventory for stable environments

Example:

```text
frontend -> employee-api.internal
salary-api -> scylla.internal
prometheus -> node-exporter.internal:9100
```

Benefits:

- Decouples clients from changing IPs
- Supports scaling
- Simplifies deployments
- Enables environment-specific endpoints
- Reduces manual configuration

---

# 15. DNS-based discovery vs load-balancer discovery

| DNS name to instance | DNS name to load balancer |
|---|---|
| Client may connect directly to one host | LB distributes traffic |
| More operational responsibility | Health checks and balancing available |
| IP changes may matter | Backend changes hidden behind LB |
| Useful for internal fixed services | Better for scalable applications |

For production microservices, prefer a stable service endpoint rather than individual ephemeral instance IPs.

---

# 16. Kubernetes service discovery

Kubernetes commonly provides DNS names such as:

```text
service-name.namespace.svc.cluster.local
```

Examples:

```text
employee-api.default.svc.cluster.local
```

Within the same namespace, the short name may work:

```bash
curl http://employee-api:8080/health
```

Typical flow:

```text
Pod -> Cluster DNS -> Service IP -> Endpoint Pod
```

Important objects:

- Service
- Endpoints or EndpointSlices
- CoreDNS
- Namespace
- ClusterIP
- Headless Service

A Service gives clients a stable virtual endpoint while backend Pods change.

---

# 17. Headless Services

A headless Service uses:

```yaml
spec:
  clusterIP: None
```

Instead of one virtual ClusterIP, DNS can return Pod IPs or endpoint records.

Useful for:

- Stateful systems
- Database clusters
- Peer discovery
- Systems requiring individual node identity

Do not assume every service should be headless. Normal ClusterIP Services are generally preferable for ordinary application traffic.

---

# 18. SRV records

SRV records describe a service, protocol, priority, weight, port, and target.

Example query:

```bash
dig SRV _http._tcp.example.com
```

Format:

```text
_service._protocol.name
```

SRV records are useful when clients need to discover both hostname and port.

---

# 19. DNS debugging scenarios

## Scenario A: `Could not resolve host`

Run:

```bash
getent hosts api.internal
resolvectl status
cat /etc/resolv.conf
dig api.internal
```

Check:

- DNS server reachable
- Correct search domain
- Private zone association
- Record exists
- Resolver configuration
- VPN/VPC connectivity
- `/etc/hosts` overrides

## Scenario B: DNS returns the wrong IP

Check:

```bash
getent hosts app.example.com
dig app.example.com
cat /etc/hosts
dig @authoritative-server app.example.com
```

Possible causes:

- Cache
- Stale hosts entry
- Split-horizon DNS
- Wrong hosted zone
- CNAME chain
- Incorrect record update

## Scenario C: DNS works on laptop but not EC2

Compare from both locations:

```bash
hostname
cat /etc/resolv.conf
resolvectl status
dig app.internal
```

Check:

- VPC DNS settings
- Private hosted-zone association
- Route/NAT/VPN
- Security controls
- Search domain
- Resolver address

## Scenario D: API still reaches old server after DNS change

Check:

```bash
dig app.example.com
curl -v https://app.example.com
```

Possible causes:

- Resolver cache
- Application DNS cache
- NGINX upstream behavior
- Existing keep-alive connections
- Local hosts file
- CDN or proxy cache

## Scenario E: Kubernetes service name fails

Check:

```bash
kubectl get svc
kubectl get endpoints
kubectl get endpointslices
kubectl get pods -o wide
kubectl exec -it POD -- nslookup employee-api
kubectl exec -it POD -- cat /etc/resolv.conf
```

Investigate:

- Service name/namespace
- CoreDNS
- Service selector
- Ready endpoints
- NetworkPolicy
- Port and targetPort

---

# 20. DNS security considerations

- Use trusted resolvers.
- Protect private hosted zones.
- Avoid exposing internal names publicly.
- Monitor unexpected record changes.
- Use DNSSEC where appropriate for public zones.
- Avoid embedding secrets in TXT records.
- Restrict permissions to modify DNS.
- Audit Route 53 changes.
- Validate DNS responses in security-sensitive workflows.

DNS is not an authentication mechanism. A resolved name may still point to an unauthorized or compromised endpoint.

---

# 21. Practical command reference

```bash
getent hosts NAME
resolvectl query NAME
resolvectl status
cat /etc/resolv.conf
cat /etc/hosts
cat /etc/nsswitch.conf
dig NAME
dig +short NAME
dig A NAME
dig AAAA NAME
dig CNAME NAME
dig SRV _http._tcp.NAME
dig -x IP
dig +trace NAME
nc -vz NAME PORT
curl -v http://NAME:PORT/health
```

---

# 22. Interview questions

1. What is DNS?
2. Explain A, AAAA, CNAME, MX, TXT, NS, PTR, and SRV records.
3. What is the difference between authoritative and recursive DNS?
4. What is TTL?
5. Why can users still reach an old load balancer after a DNS change?
6. What is split-horizon DNS?
7. How do you troubleshoot `Could not resolve host`?
8. Why can DNS work on a laptop but fail on EC2?
9. What is the difference between DNS failure and TCP failure?
10. How does Route 53 private hosted-zone resolution work?
11. Compare DNS routing with load-balancer routing.
12. What is service discovery?
13. Why should applications avoid hardcoded EC2 IPs?
14. How does Kubernetes DNS work?
15. What is a headless Service?
16. What are SRV records?
17. How would you troubleshoot a Kubernetes Service DNS failure?
18. Why can NGINX continue using an old backend IP?
19. How do Jenkins and Ansible execution environments affect DNS?
20. Is DNS a security or authentication mechanism?

---

# 23. Interview checklist

- [ ] Explain DNS resolution.
- [ ] Know common DNS record types.
- [ ] Use `dig`, `getent`, and `resolvectl`.
- [ ] Explain TTL and caching.
- [ ] Distinguish authoritative and recursive DNS.
- [ ] Explain `/etc/hosts` and `/etc/resolv.conf`.
- [ ] Troubleshoot private EC2 DNS.
- [ ] Explain Route 53 private hosted zones.
- [ ] Explain split-horizon DNS.
- [ ] Explain DNS-based service discovery.
- [ ] Understand Kubernetes Service DNS.
- [ ] Troubleshoot DNS from Jenkins/Ansible.
- [ ] Explain why DNS does not prove service health.

---

# Key DevOps principle

> **Use stable service names instead of hardcoded infrastructure IPs, but always validate the complete path from name resolution to application health.**
