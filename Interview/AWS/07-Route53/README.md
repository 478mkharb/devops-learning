# Route 53

### Q1. What is Amazon Route 53?

**Answer:** Amazon Route 53 is AWS's highly available and scalable **DNS service**. It provides authoritative DNS hosting, domain registration, DNS health checks, and DNS routing policies.

---

### Q2. What is a hosted zone?

**Answer:** A hosted zone is a container for DNS records for a domain.

There are two main types:

| Hosted Zone | Purpose |
|---|---|
| Public hosted zone | Resolves DNS names on the public Internet |
| Private hosted zone | Resolves DNS names within associated VPCs |

---

### Q3. What is a public hosted zone?

**Answer:** A public hosted zone contains DNS records that are publicly resolvable on the Internet. Route 53 provides authoritative name servers for the zone, and those name servers are delegated to by the domain's parent DNS zone.

---

### Q4. What is a private hosted zone?

**Answer:** A private hosted zone contains DNS records that can be resolved from associated VPCs. It is commonly used for internal application names such as:

```text
api.internal.example.com
db.internal.example.com
```

The VPC must have DNS resolution and DNS hostnames appropriately enabled for normal private DNS resolution.

---

### Q5. What is an authoritative DNS server?

**Answer:** An authoritative DNS server stores the definitive DNS records for a DNS zone and provides authoritative answers for names within that zone.

In Route 53, the name servers assigned to a public hosted zone are authoritative for that zone.

---

### Q6. What is a recursive DNS resolver?

**Answer:** A recursive DNS resolver receives DNS queries from clients and obtains answers on their behalf. If the answer is not already cached, it can query the DNS hierarchy, such as the root servers, TLD servers, and authoritative name servers.

```text
Client
  ↓
Recursive Resolver
  ↓
Root
  ↓
TLD
  ↓
Authoritative DNS
  ↓
IP address
```

---

### Q7. What is TTL in DNS?

**Answer:** TTL (Time To Live) specifies how long a DNS resolver can cache a DNS response before it should query the authoritative DNS server again.

For example:

```text
www.example.com → 192.0.2.44
TTL = 300 seconds
```

A resolver can normally cache that answer for up to 300 seconds.

---

### Q8. What is an A record?

**Answer:** An A record maps a DNS name to an **IPv4 address**.

Example:

```text
www.example.com → 192.0.2.44
```

---

### Q9. What is an AAAA record?

**Answer:** An AAAA record maps a DNS name to an **IPv6 address**.

Example:

```text
www.example.com → 2001:db8::10
```

---

### Q10. What is a CNAME record?

**Answer:** A CNAME record maps one DNS name to another DNS name.

Example:

```text
www.example.com → app.example.net
```

The DNS resolver then resolves `app.example.net` to its final address.

A CNAME cannot normally be used at the zone apex.

---

### Q11. What is an Alias record in Route 53?

**Answer:** A Route 53 Alias record is an AWS-specific DNS record that can point a DNS name to selected AWS resources or another supported Route 53 resource without requiring the target's IP address to be specified.

Common targets include:

- Application Load Balancer
- Network Load Balancer
- CloudFront distribution
- API Gateway
- S3 website endpoint
- Another Route 53 record

Unlike a CNAME, an Alias record can be used at the **zone apex**.

---

### Q12. What is the difference between an Alias record and a CNAME?

**Answer:**

| Alias | CNAME |
|---|---|
| Route 53-specific | Standard DNS record type |
| Can point to supported AWS resources | Points to another DNS name |
| Can be used at the zone apex | Cannot normally be used at the zone apex |
| Route 53 does not charge for Alias queries | Standard DNS behavior applies |
| Can be used for supported AWS resources such as ALB and CloudFront | Can point to DNS names |

A common AWS example is:

```text
example.com → ALB
```

using an Alias record.

---

### Q13. What is an MX record?

**Answer:** An MX (Mail Exchange) record identifies the mail servers responsible for receiving email for a domain. MX records include a **priority** value.

Lower numeric priority values are preferred over higher values.

---

### Q14. What is a TXT record?

**Answer:** A TXT record stores text associated with a DNS name. Common uses include:

- Domain ownership verification
- SPF-related email configuration
- DKIM-related configuration
- DMARC-related information

---

### Q15. What is an NS record?

**Answer:** An NS (Name Server) record identifies the authoritative name servers for a DNS zone.

For a public Route 53 hosted zone, the assigned Route 53 name servers are published through DNS delegation so resolvers know where to obtain authoritative answers.

---

### Q16. What is an SOA record?

**Answer:** An SOA (Start of Authority) record contains administrative and timing information about a DNS zone, including the authoritative name server and other zone parameters.

---

### Q17. What is an SRV record?

**Answer:** An SRV (Service) record specifies the location of a service using fields such as:

- Priority
- Weight
- Port
- Target hostname

It is commonly used for service discovery.

---

### Q18. Can a CNAME be used at the zone apex?

**Answer:** No. A CNAME cannot normally be created at the zone apex, such as:

```text
example.com
```

because the zone apex must contain required DNS records such as NS and SOA.

In Route 53, an **Alias record** can be used at the zone apex to point to supported AWS resources such as an ALB or CloudFront distribution.

---

### Q19. What is simple routing?

**Answer:** Simple routing is used when a DNS name has a straightforward DNS answer. It does not distribute traffic using weights, latency, geography, or health-based failover.

For example:

```text
www.example.com → 192.0.2.44
```

It is suitable for a basic single-endpoint configuration.

---

### Q20. What is weighted routing?

**Answer:** Weighted routing assigns a relative weight to multiple records and uses those weights when selecting responses.

Example:

```text
Version A → Weight 90
Version B → Weight 10
```

This can be used for:

- Gradual migrations
- Canary deployments
- Traffic splitting
- Testing a new application version

Weighted routing is based on **configured weights**, not the actual number of active users.

---

### Q21. What is latency-based routing?

**Answer:** Latency-based routing routes DNS queries to the AWS Region that Route 53 determines provides the lowest latency among the configured resources.

Example:

```text
User
 ├── us-east-1 endpoint
 ├── eu-west-1 endpoint
 └── ap-south-1 endpoint
```

Route 53 selects the Region expected to provide the lowest latency for that user.

---

### Q22. What is failover routing?

**Answer:** Failover routing provides an **active-passive** DNS configuration with a primary and secondary record.

When the primary is considered unhealthy, Route 53 can return the secondary record.

```text
Primary
   ↓
Healthy → Primary returned

Unhealthy
   ↓
Secondary returned
```

It is commonly used for disaster recovery.

---

### Q23. What is geolocation routing?

**Answer:** Geolocation routing selects a DNS response based on the geographic location associated with the DNS query, such as:

- Continent
- Country
- US state

It is useful when different users should receive different endpoints based on geography.

---

### Q24. What is geoproximity routing?

**Answer:** Geoproximity routing routes traffic based on the geographic location of resources and users. It can use **bias** to expand or shrink the geographic area served by a resource.

It is useful when you want geographic control over traffic distribution rather than simply matching users to fixed country or continent rules.

---

### Q25. What is IP-based routing?

**Answer:** IP-based routing lets Route 53 select a DNS record based on the **source IP address of the DNS query** and configured CIDR collections.

It can be useful when different client networks need to resolve to different endpoints.

---

### Q26. What is multivalue answer routing?

**Answer:** Multivalue answer routing allows Route 53 to return multiple values for a DNS name, with Route 53 able to return only healthy values when health checks are configured.

It can improve availability for simple DNS-based endpoint selection, but it is **not a replacement for a load balancer**.

---

### Q27. How do weighted and latency-based routing differ?

**Answer:**

| Weighted Routing | Latency-Based Routing |
|---|---|
| Uses configured relative weights | Uses estimated network latency |
| Controls traffic proportions | Selects the lowest-latency Region |
| Useful for traffic splitting and migrations | Useful for multi-Region applications |
| Example: 90% / 10% | Example: nearest/lowest-latency Region |

In short:

> **Weighted routing controls how much traffic goes to each endpoint; latency routing chooses the endpoint expected to provide the lowest latency.**

---

### Q28. How does failover routing use health checks?

**Answer:** Failover routing uses primary and secondary records. When health checks are associated with the appropriate record, Route 53 can determine whether the primary endpoint is healthy.

If the primary is unhealthy, Route 53 can return the secondary endpoint.

---

### Q29. What is a Route 53 health check?

**Answer:** A Route 53 health check monitors the health of an endpoint or evaluates the status of other health checks.

For an endpoint health check, Route 53 can check supported protocols such as:

- HTTP
- HTTPS
- TCP

For HTTP/HTTPS checks, you can specify settings such as the port, path, and expected response criteria.

Health-check status can be used by supported Route 53 routing policies.

---

### Q30. Can a Route 53 health check monitor another health check?

**Answer:** Yes. Route 53 supports **calculated health checks**, which combine the status of multiple child health checks using logical evaluation.

This allows a routing decision to depend on the combined health of multiple endpoints.

---

### Q31. Does a Route 53 health check automatically monitor an ALB target?

**Answer:** No. An ALB has its own target health-check mechanism for determining whether registered targets are healthy.

A Route 53 health check is a separate DNS-level health-check mechanism.

For example:

```text
Route 53 Health Check
        ↓
   ALB DNS name
        ↓
       ALB
        ↓
   ALB Target Health Checks
        ↓
      EC2 targets
```

Do not confuse **Route 53 health checks** with **ELB target health checks**.

---

### Q32. How does DNS caching affect a Route 53 record change?

**Answer:** Recursive DNS resolvers cache DNS responses according to their TTL.

If a Route 53 record changes while a resolver still has the old answer cached, clients using that resolver can continue receiving the old value until the cached TTL expires.

```text
Route 53
Old IP → New IP

Resolver still has:
Old IP

        ↓
Clients may still reach old endpoint
        ↓
Cache expires
        ↓
Resolver queries authoritative DNS
        ↓
New IP
```

---

### Q33. Why might users temporarily reach an old endpoint after a DNS update?

**Answer:** The old DNS answer may still be cached by recursive resolvers or local DNS caches.

For example:

```text
TTL = 300 seconds
```

If a resolver cached the old address shortly before the record was changed, it can continue returning that old address until the cached entry expires.

This is why lowering the TTL **before** a planned DNS migration can reduce the duration of stale cached answers.

---

### Q34. What happens when a DNS resolver does not have a cached answer?

**Answer:** The recursive resolver performs DNS resolution through the DNS hierarchy.

Conceptually:

```text
Client
  ↓
Recursive Resolver
  ↓
Root DNS
  ↓
TLD DNS (.com)
  ↓
Authoritative DNS
  ↓
Route 53
  ↓
IP address
```

The resolver then returns the answer to the client and normally caches it according to the TTL.

---

### Q35. What is DNS delegation?

**Answer:** DNS delegation is the process by which a parent DNS zone identifies the authoritative name servers responsible for a child zone.

For example:

```text
Root
  ↓
.com
  ↓
example.com
  ↓
Route 53 authoritative name servers
```

For a public Route 53 hosted zone, the domain's delegation must point to the Route 53 name servers assigned to that hosted zone.

---

### Q36. What is the difference between domain registration and DNS hosting in Route 53?

**Answer:** They are separate functions.

| Domain Registration | DNS Hosting |
|---|---|
| Registers/maintains the domain name | Stores and serves DNS records |
| Example: registering `example.com` | Example: A record for `www.example.com` |
| Involves a registrar | Uses a Route 53 hosted zone |

A domain can be registered through one registrar while its DNS is hosted by Route 53 or another DNS provider.

---

### Q37. Can Route 53 route traffic directly to an EC2 instance?

**Answer:** Yes. Route 53 can return an EC2 instance's public IP through an A record or use an appropriate DNS configuration.

However, directly pointing production DNS to an individual EC2 public IP is often less resilient than pointing DNS to a load balancer or another highly available endpoint.

For AWS resources that support Alias records, an Alias is generally preferred over hard-coding an address when appropriate.

---

### Q38. Can Route 53 point to an Application Load Balancer?

**Answer:** Yes. A Route 53 Alias record can point a DNS name to an Application Load Balancer.

Example:

```text
www.example.com
       ↓
Alias record
       ↓
ALB
       ↓
Target Group
       ↓
EC2
```

This is a common production architecture.

---

### Q39. What is the difference between Route 53 routing and a load balancer?

**Answer:** Route 53 makes a **DNS-level decision** about which DNS response to return. A load balancer makes a **connection/request-level decision** about which backend target should receive traffic.

```text
Route 53
DNS decision
      ↓
ALB
Traffic distribution
      ↓
Target Group
      ↓
EC2
```

Route 53 can direct users toward a Region or endpoint, while an ALB can distribute requests among healthy targets.

---

### Q40. Does Route 53 send application traffic?

**Answer:** No. Route 53 is a DNS service.

It normally returns DNS information such as an IP address or DNS target. The client then connects directly to the returned endpoint.

```text
Client
  ↓
Route 53
  ↓
DNS answer
  ↓
Client
  ↓
ALB / CloudFront / Application
```

Route 53 is therefore involved in **name resolution**, not in carrying the application's HTTP/HTTPS payload.

---

### Q41. What is Route 53 private DNS resolution used for?

**Answer:** Route 53 private hosted zones provide internal DNS names that resolve within associated VPCs.

For example:

```text
api.internal.example.com
        ↓
10.0.2.50
```

This is useful for internal application communication without exposing the DNS name publicly.

---

### Q42. Can a private hosted zone and public hosted zone use the same domain name?

**Answer:** Yes. You can have a public hosted zone and a private hosted zone with the same domain name, with the private hosted zone associated with VPCs.

For associated VPCs, DNS resolution can use the private hosted zone, allowing internal resources to resolve names differently from public users.

This pattern is commonly called **split-horizon DNS**.

---

### Q43. What is split-horizon DNS?

**Answer:** Split-horizon DNS means the same DNS name can resolve to different answers depending on where the DNS query originates.

Example:

```text
Internet user
www.example.com
      ↓
Public IP / CloudFront / ALB

Internal VPC user
www.example.com
      ↓
Private IP / internal ALB
```

It is commonly implemented using public and private DNS zones.

---

### Q44. What is a Route 53 Resolver?

**Answer:** Route 53 Resolver is the DNS resolution service associated with Amazon VPC. It provides DNS resolution for VPC resources and supports forwarding DNS queries between VPCs, on-premises networks, and other DNS environments using Resolver endpoints and rules.

---

### Q45. What is a Route 53 Resolver inbound endpoint?

**Answer:** A Resolver inbound endpoint allows DNS queries from external networks, such as on-premises networks, to be sent into the VPC for resolution by Route 53 Resolver.

Example:

```text
On-Premises DNS
      ↓
VPN / Direct Connect
      ↓
Resolver Inbound Endpoint
      ↓
Route 53 Resolver
      ↓
Private DNS
```

---

### Q46. What is a Route 53 Resolver outbound endpoint?

**Answer:** A Resolver outbound endpoint allows DNS queries originating in a VPC to be forwarded to DNS servers outside the VPC, such as on-premises DNS servers.

Example:

```text
EC2
 ↓
Route 53 Resolver
 ↓
Outbound Endpoint
 ↓
VPN / Direct Connect
 ↓
On-Premises DNS
```

---

### Q47. What is a Route 53 Resolver rule?

**Answer:** A Resolver rule defines how DNS queries matching specified domain names should be handled.

For example:

```text
corp.example.com
       ↓
Forward to on-premises DNS
```

Resolver rules are useful when VPC workloads need to resolve names hosted by external or on-premises DNS systems.

---

### Q48. What is DNS failover versus application failover?

**Answer:** Route 53 failover is **DNS-level failover**. It changes the DNS answer returned to clients based on configured health conditions.

It does not instantly move an existing TCP connection or HTTP session from one server to another.

For example:

```text
Route 53
Primary unhealthy
      ↓
Return secondary DNS answer
      ↓
New client connections use secondary
```

Existing connections are not magically transferred by Route 53.

---

### Q49. What is a common Route 53 multi-Region architecture?

**Answer:** A common architecture uses Route 53 to direct users to application endpoints in multiple AWS Regions.

For example, latency-based routing can select between:

```text
                Route 53
               /        \
              /          \
       us-east-1       ap-south-1
          ALB              ALB
           ↓                ↓
        Targets          Targets
```

This can provide lower latency and improve regional resilience.

---

### Q50. What is the difference between latency, geolocation, and geoproximity routing?

**Answer:**

| Policy | Primary decision |
|---|---|
| Latency-based | Lowest estimated network latency |
| Geolocation | Geographic location of the DNS query |
| Geoproximity | Geographic distance between users/resources, adjustable with bias |

Use **latency routing** when network performance is the primary concern, **geolocation** when location-based behavior is required, and **geoproximity** when you need geographic traffic distribution with adjustable boundaries.
