# Route 53

### Q1. What is Route 53?

**Answer:** Amazon Route 53 is AWS's managed DNS service. It provides authoritative DNS hosting, domain registration, health checks, and several DNS routing policies.

---

### Q2. What is a hosted zone?

**Answer:** A hosted zone is a container for DNS records for a domain. A public hosted zone serves Internet DNS queries, while a private hosted zone provides DNS resolution within associated VPCs.

---

### Q3. What is a public hosted zone?

**Answer:** A public hosted zone contains DNS records that Route 53 publishes for Internet DNS resolution. Its records can be queried by clients on the public Internet through the domain's authoritative name servers.

---

### Q4. What is a private hosted zone?

**Answer:** A private hosted zone contains DNS records that are resolvable only from associated VPCs using Amazon VPC DNS resolution. It is commonly used for internal application names and private service discovery.

---

### Q5. What is an authoritative DNS server?

**Answer:** An authoritative DNS server stores the definitive DNS records for a domain's zone and answers queries for names in that zone. Route 53 hosted zones are served by Route 53 authoritative name servers.

---

### Q6. What is TTL?

**Answer:** DNS TTL specifies how long a recursive DNS resolver can cache an answer before it needs to query an authoritative server again.

---

### Q7. What is an A record?

**Answer:** An A record maps a DNS name to an IPv4 address.

---

### Q8. What is an AAAA record?

**Answer:** An AAAA record maps a DNS name to an IPv6 address.

---

### Q9. What is a CNAME record?

**Answer:** A CNAME record maps one DNS name to another DNS name. It cannot normally be used at the zone apex.

---

### Q10. What is an Alias record?

**Answer:** A Lambda alias is a named pointer to a published Lambda function version. It lets applications use a stable name such as production while the underlying version changes, and it can support controlled traffic shifting between versions.

---

### Q11. What is an MX record?

**Answer:** An MX record identifies the mail servers responsible for receiving email for a domain and includes a priority value.

---

### Q12. What is a TXT record?

**Answer:** A TXT record stores text associated with a DNS name and is commonly used for domain verification and email-security mechanisms.

---

### Q13. What is an NS record?

**Answer:** An NS record identifies the authoritative name servers for a DNS zone.

---

### Q14. What is an SOA record?

**Answer:** An SOA record contains authoritative information about a DNS zone, including the primary name server and zone timing information.

---

### Q15. What is an SRV record?

**Answer:** An SRV record identifies the location of a service using priority, weight, port, and target hostname.

---

### Q16. Can a CNAME be used at the zone apex?

**Answer:** A CNAME cannot normally be created at the zone apex, such as `example.com`, because the apex must contain the zone's required NS and SOA records. In Route 53, an Alias record can point the apex to supported AWS resources such as an ALB or CloudFront distribution.

---

### Q17. What is simple routing?

**Answer:** Simple routing returns a straightforward DNS answer without distributing traffic using weights, latency, or geographic rules. It is suitable for a basic single-endpoint setup.

---

### Q18. What is weighted routing?

**Answer:** Weighted routing assigns relative weights to multiple records and uses those weights when selecting DNS responses. It is useful for traffic splitting and gradual migrations.

---

### Q19. What is latency-based routing?

**Answer:** Latency-based routing directs clients to the Region that Route 53 determines provides the lowest latency among configured endpoints.

---

### Q20. What is failover routing?

**Answer:** Failover routing uses primary and secondary records and health-check status to return the appropriate endpoint. It is commonly used for active-passive disaster recovery.

---

### Q21. What is geolocation routing?

**Answer:** Geolocation routing selects a DNS record based on the geographic location associated with the DNS query, such as country or continent.

---

### Q22. What is geoproximity routing?

**Answer:** Geoproximity routing routes based on the geographic location of resources and users and can use bias to expand or shrink the area served by a resource.

---

### Q23. What is IP-based routing?

**Answer:** IP-based routing selects a DNS record based on the source IP address and configured CIDR mappings.

---

### Q24. What is multivalue answer routing?

**Answer:** Multivalue answer routing returns multiple healthy values for a DNS name. It can improve availability for simple endpoint selection but is not a replacement for a load balancer.

---

### Q25. How do weighted and latency routing differ?

**Answer:** Weighted routing distributes DNS responses according to configured relative weights, making it useful for traffic splitting and gradual migrations. Latency-based routing selects the endpoint associated with the lowest measured latency for the requester. Weighted routing is about configured proportions; latency routing is about network latency.

---

### Q26. How does failover routing use health checks?

**Answer:** A failover routing policy identifies a primary and secondary record. Route 53 uses health-check status for the primary, when configured, and returns the primary while it is healthy; when it is unhealthy, DNS responses can fail over to the secondary.

---

### Q27. What is a Route 53 health check?

**Answer:** A load-balancer health check periodically tests a target using configured protocol, port, path, and success criteria. Traffic is normally sent only to targets considered healthy.

---

### Q28. Can health checks monitor endpoints?

**Answer:** Yes. Route 53 health checks can monitor an endpoint such as a web server, or they can evaluate the status of other health checks. Health-check results can be used by supported routing policies.

---

### Q29. Can one health check monitor other health checks?

**Answer:** Yes. Route 53 supports calculated health checks that combine the status of multiple health checks using logical evaluation. This allows routing decisions to depend on the combined health of several endpoints.

---

### Q30. How does DNS caching affect a record change?

**Answer:** Recursive DNS resolvers cache DNS answers according to their TTL. After a record changes in Route 53, users whose resolvers still have the old answer can continue reaching the old endpoint until the cached record expires.

---

### Q31. Why might users temporarily resolve an old endpoint after a DNS update?

**Answer:** Because recursive resolvers and client-side DNS caches may still contain the old record. They continue using the cached answer until its TTL expires and they query an authoritative DNS server again.

---
