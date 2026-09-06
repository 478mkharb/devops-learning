# Route 53

[⬅️ Back to AWS Topics](../README.md)

## 🔑 Keywords

🌍 DNS | A | AAAA | CNAME | Alias | TTL | Health Check | Weighted | Latency | Failover | Geo

## 🧠 Core Memory

🧠 **Remember:** **Routing policy = which answer**, **TTL = how long resolver caches it**, **Health Check = whether endpoint is healthy**.

---

## ❓ Interview Questions

### 📌 DNS

#### Q1. What is Route 53?

**💡 Answer:** Amazon Route 53 is AWS's managed DNS service. It provides authoritative DNS hosting, domain registration, health checks, and routing policies.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q2. What is a hosted zone?

**💡 Answer:** A hosted zone is a container for DNS records for a domain. A public hosted zone serves Internet DNS queries; a private hosted zone serves DNS resolution inside associated VPCs.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q3. What is a public hosted zone?

**💡 Answer:** A hosted zone is a container for DNS records for a domain. A public hosted zone serves Internet DNS queries; a private hosted zone serves DNS resolution inside associated VPCs.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q4. What is a private hosted zone?

**💡 Answer:** A hosted zone is a container for DNS records for a domain. A public hosted zone serves Internet DNS queries; a private hosted zone serves DNS resolution inside associated VPCs.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q5. What is an authoritative DNS server?

**💡 Answer:** Explain the DNS record or routing policy, how Route 53 selects the answer, and how TTL/health checks affect client behavior.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q6. What is TTL?

**💡 Answer:** DNS TTL specifies how long a resolver may cache a DNS answer before querying again. A lower TTL can make changes visible sooner but increases DNS query traffic.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Record Types

#### Q7. What is an A record?

**💡 Answer:** An A record maps a DNS name to an IPv4 address.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q8. What is an AAAA record?

**💡 Answer:** An A record maps a DNS name to an IPv4 address.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q9. What is a CNAME record?

**💡 Answer:** A CNAME record maps a DNS name to another DNS name. It cannot generally be used at the zone apex; Route 53 Alias records are used for supported AWS targets and apex records.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q10. What is an Alias record?

**💡 Answer:** A Route 53 Alias record maps a name to supported AWS resources or another supported Route 53 target without requiring a CNAME at the zone apex. It is AWS-specific and does not incur a Route 53 query charge for alias queries to AWS resources.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q11. What is an MX record?

**💡 Answer:** An MX record identifies mail servers responsible for receiving email for a domain.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q12. What is a TXT record?

**💡 Answer:** A TXT record stores text associated with a DNS name. It is commonly used for domain verification and email-security mechanisms such as SPF-related records and DKIM data.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q13. What is an NS record?

**💡 Answer:** An NS record identifies the authoritative name servers for a DNS zone.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q14. What is an SOA record?

**💡 Answer:** An A record maps a DNS name to an IPv4 address.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q15. What is an SRV record?

**💡 Answer:** An SRV record specifies the location of a service using a priority, weight, port, and target hostname.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q16. Can a CNAME be used at the zone apex?

**💡 Answer:** A CNAME record maps a DNS name to another DNS name. It cannot generally be used at the zone apex; Route 53 Alias records are used for supported AWS targets and apex records.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Routing Policies

#### Q17. What is simple routing?

**💡 Answer:** Simple routing returns a single resource or set of values without weighting or latency-based selection. It is suitable when straightforward DNS resolution is sufficient.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q18. What is weighted routing?

**💡 Answer:** Weighted routing assigns relative weights to records and distributes DNS responses according to those weights. It is useful for traffic splitting, testing, and gradual migrations.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q19. What is latency-based routing?

**💡 Answer:** Latency-based routing sends users to the AWS Region that Route 53 determines provides the lowest latency among configured records.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q20. What is failover routing?

**💡 Answer:** Failover routing uses primary and secondary records and health checks to return the healthy endpoint. It is commonly used for active-passive disaster recovery.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q21. What is geolocation routing?

**💡 Answer:** Geolocation routing selects a record based on the geographic location from which the DNS query originates, such as country or continent.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q22. What is geoproximity routing?

**💡 Answer:** Geoproximity routing routes based on the geographic location of resources and users and can use bias to expand or shrink the geographic area served by a resource.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q23. What is IP-based routing?

**💡 Answer:** IP-based routing selects a Route 53 record based on the source IP address and configured CIDR mappings.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q24. What is multivalue answer routing?

**💡 Answer:** Multivalue answer routing returns multiple healthy values and can be used to improve availability when clients can select among returned endpoints. It is not a replacement for a load balancer.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q25. How do weighted and latency routing differ?

**💡 Answer:** Explain the DNS record or routing policy, how Route 53 selects the answer, and how TTL/health checks affect client behavior.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q26. How does failover routing use health checks?

**💡 Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

### 📌 Health & Operations

#### Q27. What is a Route 53 health check?

**💡 Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q28. Can health checks monitor endpoints?

**💡 Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q29. Can one health check monitor other health checks?

**💡 Answer:** A load-balancer target health check periodically tests a configured protocol, port, and path or connection behavior. Only healthy targets receive traffic.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q30. How does DNS caching affect a record change?

**💡 Answer:** An A record maps a DNS name to an IPv4 address.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

#### Q31. Why might users temporarily resolve an old endpoint after a DNS update?

**💡 Answer:** Explain the DNS record or routing policy, how Route 53 selects the answer, and how TTL/health checks affect client behavior.

**🔑 Keywords:** `Route` · `AWS` · `Interview`

**⚡ Interview Tip:** Start with the definition, explain the behavior, then give the AWS use case or comparison. For scenario questions, state why this option is preferable and mention the key trade-off.

---

## 🚀 Last-Minute Revision

> 🧠 **Remember:** **Routing policy = which answer**, **TTL = how long resolver caches it**, **Health Check = whether endpoint is healthy**.

[⬆️ Back to top](#route-53)

[⬅️ Back to AWS Topics](../README.md)