# VPC & AWS Networking

### Q1. What is a VPC?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and network security controls. A VPC can connect to the Internet, other VPCs, AWS services, or on-premises networks.

---

### Q2. What is a CIDR block?

**Answer:** A CIDR block defines an IP address range using a network address and prefix length.

For example:

```text
10.0.0.0/16
```

The `/16` indicates that the first 16 bits represent the network portion of the address.

---

### Q3. What is a subnet?

**Answer:** A subnet is an IP address range within a VPC and exists entirely within **one Availability Zone**. A subnet is associated with a route table, which determines how traffic is routed.

---

### Q4. What is the difference between a public and private subnet?

**Answer:** The distinction is based primarily on routing.

| Public Subnet | Private Subnet |
|---|---|
| Has a route to an Internet Gateway | Does not have a direct route to an Internet Gateway |
| Resources can have public IPv4 addresses and Internet connectivity when other requirements are satisfied | Resources normally use private IP addresses |
| Common for ALBs, NAT Gateways, bastions | Common for application servers and databases |
| Outbound Internet can use the IGW directly | Outbound Internet can use a NAT Gateway if required |

A subnet is **not public simply because an instance has a public IP**; the subnet's route table must provide a path to an Internet Gateway.

---

### Q5. What is a route table?

**Answer:** A route table contains rules that map a **destination** to a **target**. It determines where traffic leaving resources in associated subnets should be forwarded.

Example:

```text
Destination     Target
10.0.0.0/16     local
0.0.0.0/0       igw-xxxx
10.1.0.0/16     pcx-xxxx
```

When multiple routes match, AWS uses the **most specific matching route**.

---

### Q6. What is a route?

**Answer:** A route is an individual destination-to-target rule in a route table.

For example:

```text
Destination: 10.1.0.0/16
Target:      pcx-123456
```

This tells AWS to send traffic destined for `10.1.0.0/16` through the VPC peering connection.

---

### Q7. What is an Internet Gateway?

**Answer:** An Internet Gateway (IGW) is a horizontally scaled, redundant VPC component that provides Internet connectivity for resources with appropriate routing and public addressing. It is attached to the VPC and can support both inbound and outbound Internet traffic when the applicable routing and security controls allow it.

---

### Q8. What is a NAT Gateway?

**Answer:** A NAT Gateway enables resources in private subnets to initiate connections to destinations outside the VPC, such as the public Internet, without allowing unsolicited inbound connections to those private resources.

For IPv4 Internet access, a public NAT Gateway is normally placed in a public subnet and routes traffic through an Internet Gateway.

---

### Q9. Why does a private subnet need a NAT Gateway for outbound Internet access?

**Answer:** A private subnet has no direct route to an Internet Gateway. If its resources need outbound IPv4 Internet access, the route table can send traffic to a NAT Gateway in a public subnet. The NAT Gateway translates the private source address and forwards the traffic through the Internet Gateway.

```text
Private EC2
    ↓
Private Route Table
    ↓
NAT Gateway
    ↓
Internet Gateway
    ↓
Internet
```

---

### Q10. Can a NAT Gateway accept unsolicited inbound Internet traffic?

**Answer:** No. A NAT Gateway allows return traffic for connections initiated by resources behind the NAT Gateway, but it does not provide unsolicited inbound connectivity to those resources.

---

### Q11. What is VPC peering?

**Answer:** VPC peering is a private network connection between two VPCs that allows resources to communicate using private IP addresses. The VPCs must have non-overlapping CIDR ranges, and the required routes and security controls must be configured.

VPC peering is **non-transitive**.

```text
VPC-A ←→ VPC-B ←→ VPC-C

VPC-A cannot automatically communicate with VPC-C through VPC-B.
```

---

### Q12. What is required for communication between two VPCs using VPC peering?

**Answer:** You need:

1. Non-overlapping VPC CIDRs.
2. An active VPC peering connection.
3. A route to the remote VPC CIDR in the relevant route table on each side.
4. Security Groups that allow the traffic.
5. NACLs that allow the traffic when restrictive rules are configured.
6. OS and application firewalls to permit the traffic when applicable.

Example:

```text
VPC-A                              VPC-B
10.0.0.0/16                        10.1.0.0/16

10.1.0.0/16 → pcx-xxxx     ←     10.0.0.0/16 → pcx-xxxx
```

You do not necessarily need exactly two route tables; you need the appropriate peering routes in the route tables associated with the participating subnets.

---

### Q13. What is Transit Gateway?

**Answer:** AWS Transit Gateway is a regional network transit hub that can connect multiple VPCs and other supported networks, including on-premises networks through VPN or Direct Connect.

It provides centralized routing instead of requiring individual VPC peering connections between every VPC.

---

### Q14. When is Transit Gateway preferable to many VPC peerings?

**Answer:** Transit Gateway is preferable when you have many VPCs or hybrid networks that need controlled connectivity.

With VPC peering, a growing environment can require many individual connections:

```text
VPC-A ←→ VPC-B
VPC-A ←→ VPC-C
VPC-B ←→ VPC-C
...
```

With Transit Gateway:

```text
VPC-A ──┐
VPC-B ──┼── Transit Gateway
VPC-C ──┤
On-prem ┘
```

Transit Gateway route tables can then control which networks can communicate.

---

### Q15. What is a VPC endpoint?

**Answer:** A VPC endpoint provides private connectivity from resources in a VPC to supported AWS services or endpoint services without requiring a public Internet path.

The two major VPC endpoint types are:

| Type | How it works | Main use |
|---|---|---|
| Gateway endpoint | Uses route tables | S3, DynamoDB |
| Interface endpoint | Creates ENIs in subnets | AWS services, SaaS, endpoint services |

---

### Q16. What is a gateway endpoint?

**Answer:** A gateway endpoint provides private connectivity from a VPC to **Amazon S3 or DynamoDB** through route tables.

It does not create an ENI in the subnet.

Example:

```text
EC2
 ↓
Route Table
 ↓
Gateway Endpoint
 ↓
S3
```

Gateway endpoints do not have an hourly endpoint charge.

---

### Q17. What is an interface endpoint?

**Answer:** An interface endpoint is a VPC endpoint that creates **endpoint network interfaces (ENIs)** with private IP addresses in selected subnets.

It uses **AWS PrivateLink** to provide private connectivity to supported AWS services, endpoint services, and SaaS applications.

Security Groups can control traffic to the endpoint ENIs.

```text
EC2
 ↓
Interface Endpoint
 ↓
PrivateLink
 ↓
AWS Service / SaaS / Endpoint Service
```

---

### Q18. What is AWS PrivateLink?

**Answer:** AWS PrivateLink is a technology that provides private connectivity between consumers and specific services without exposing the service to the public Internet.

The consumer typically creates an **interface endpoint**, while the service provider exposes an **endpoint service**, commonly backed by a Network Load Balancer.

```text
Consumer VPC                    Provider VPC

EC2
 ↓
Interface Endpoint
 ↓
PrivateLink
 ↓
Endpoint Service
 ↓
NLB
 ↓
Application
```

PrivateLink can connect to a service outside the consumer VPC, including a service in another AWS account or a supported SaaS application, but it is **not a general-purpose Internet connection**.

---

### Q19. What is the difference between an interface endpoint and AWS PrivateLink?

**Answer:** They are related but not equivalent.

| Interface Endpoint | AWS PrivateLink |
|---|---|
| A VPC resource created by the consumer | Connectivity technology |
| Creates endpoint ENIs | Provides private service connectivity |
| Exists inside the consumer VPC | Connects consumer to a specific service |
| Uses PrivateLink | Powers interface-based private connectivity |

In simple terms:

> **Interface Endpoint is the consumer-side endpoint; PrivateLink is the technology that provides the private connection.**

---

### Q20. What is the difference between a VPC endpoint and an interface endpoint?

**Answer:** An **interface endpoint is a type of VPC endpoint**.

```text
VPC Endpoint
├── Gateway Endpoint
│   └── S3, DynamoDB
│
└── Interface Endpoint
    └── ENI + PrivateLink
```

---

### Q21. When should you use an S3/DynamoDB gateway endpoint?

**Answer:** Use a gateway endpoint when resources in a VPC need private connectivity to S3 or DynamoDB without requiring a NAT Gateway or public Internet path.

The endpoint is associated with route tables:

```text
EC2
 ↓
Route Table
 ↓
S3 Gateway Endpoint
 ↓
S3
```

Gateway endpoints have no hourly endpoint charge.

---

### Q22. What role does Private DNS play with interface endpoints?

**Answer:** Private DNS allows applications to continue using the normal AWS service hostname while DNS resolves the hostname to the private IP addresses of the interface endpoint inside the VPC.

For example:

```text
Application
    ↓
service.amazonaws.com
    ↓
Private DNS resolution
    ↓
Interface Endpoint private IP
    ↓
AWS Service
```

---

### Q23. What is a security group?

**Answer:** A security group is a **stateful virtual firewall** associated with network interfaces. It controls allowed inbound and outbound traffic using rules based on factors such as protocol, port, and source or destination.

Because it is stateful, response traffic for an allowed connection is automatically permitted.

---

### Q24. What is a network ACL?

**Answer:** A Network ACL (NACL) is a **stateless traffic filter associated with a subnet**. It evaluates numbered rules for inbound and outbound traffic.

Because it is stateless, return traffic must be explicitly permitted in the opposite direction.

---

### Q25. What is the difference between a Security Group and a NACL?

**Answer:**

| Security Group | NACL |
|---|---|
| Associated with ENIs | Associated with subnets |
| Stateful | Stateless |
| Allow rules | Allow and deny rules |
| Rules operate as a stateful firewall | Rules are evaluated in rule-number order |
| Return traffic is automatically permitted for an allowed connection | Return traffic must be explicitly allowed |

---

### Q26. Why are Security Groups stateful?

**Answer:** Security Groups are stateful because AWS tracks the state of connections. When traffic is allowed in one direction, the corresponding response traffic is automatically allowed without requiring a separate reverse-direction rule.

---

### Q27. Why are NACLs stateless?

**Answer:** NACLs are stateless because inbound and outbound traffic are evaluated independently.

For example, if inbound TCP traffic is allowed, the ephemeral response traffic must also be allowed by an appropriate outbound rule.

---

### Q28. What is VPC Flow Logs?

**Answer:** VPC Flow Logs capture metadata about IP traffic flowing to and from network interfaces, subnets, or VPCs depending on the configured flow-log scope.

They can be used for network troubleshooting, traffic analysis, and security investigations.

Flow Logs capture **traffic metadata, not packet contents**.

---

### Q29. What is a Site-to-Site VPN?

**Answer:** AWS Site-to-Site VPN provides encrypted connectivity between a VPC and an on-premises or remote network using **IPsec tunnels**.

A common architecture is:

```text
On-Premises
    ↓
Customer Gateway
    ↓
Internet
    ↓
Virtual Private Gateway / Transit Gateway
    ↓
VPC
```

---

### Q30. What is AWS Direct Connect?

**Answer:** AWS Direct Connect provides a dedicated network connection between a customer network and AWS through a Direct Connect location.

It avoids sending the connection over the public Internet and can provide more consistent network performance than Internet-based connectivity.

Direct Connect by itself does not automatically encrypt traffic.

---

### Q31. What is route propagation?

**Answer:** Route propagation allows routes learned through supported connectivity, such as a virtual private gateway, to be automatically populated into selected VPC route tables.

For example:

```text
On-Premises Network
       ↓
Site-to-Site VPN
       ↓
Virtual Private Gateway
       ↓
Route Propagation
       ↓
VPC Route Table
```

---

### Q32. What is a route table association?

**Answer:** A route table association links a subnet to a route table.

A subnet can use the routes in its associated route table to determine where traffic should be forwarded.

A route table can be associated with multiple subnets.

---

### Q33. What is a VPC DHCP option set?

**Answer:** A DHCP option set defines network configuration information that AWS provides to instances through DHCP, such as:

- Domain name
- Domain name servers
- NTP servers
- NetBIOS-related options

The VPC can have one DHCP option set associated with it at a time.

---

### Q34. What is AWS Network Firewall?

**Answer:** AWS Network Firewall is a managed, stateful network firewall service that provides traffic inspection and filtering for VPC traffic.

It can inspect traffic between network segments, to and from the Internet, or in centralized inspection architectures.

A common pattern is:

```text
Traffic
   ↓
Internet Gateway
   ↓
Network Firewall
   ↓
Application VPC
```

---

### Q35. What is a Prefix List?

**Answer:** A prefix list is a collection of CIDR blocks that can be referenced as a single network object in supported AWS resources.

There are two main types:

| Type | Managed by | Example |
|---|---|---|
| AWS-managed prefix list | AWS | Prefixes for supported AWS services |
| Customer-managed prefix list | Customer | Corporate network CIDRs |

A prefix list itself is **not a firewall**; it is a reusable collection of network prefixes.

---

### Q36. What is the difference between a route table destination and target?

**Answer:** The **destination** specifies where the traffic is going, while the **target** specifies the next-hop resource used to forward that traffic.

Example:

```text
Destination     Target
10.1.0.0/16     pcx-xxxx
```

This means traffic destined for `10.1.0.0/16` should be sent through the VPC peering connection.

---

### Q37. Why doesn't a VPC route table have a source column?

**Answer:** VPC route selection is primarily **destination-based**.

AWS examines the packet's destination and selects the most specific matching route. The source is not used as a normal route-table matching criterion.

This differs from Security Groups and NACLs, where source or destination addresses are part of traffic rules.

---

### Q38. Can an interface endpoint provide general Internet access?

**Answer:** No. An interface endpoint provides private connectivity to a **specific supported service or endpoint service**. It is not a replacement for a NAT Gateway or Internet Gateway.

For general Internet access from a private IPv4 subnet:

```text
EC2
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

### Q39. If I have three EC2 instances, do I need three interface endpoints?

**Answer:** No. Interface endpoints are generally created based on the **services that need private connectivity**, not the number of EC2 instances.

Multiple EC2 instances in the same VPC can use the same interface endpoint.

For high availability, an interface endpoint can be configured in multiple Availability Zones. AWS creates endpoint ENIs in the selected subnets.

---

### Q40. Can PrivateLink connect to a service outside my VPC?

**Answer:** Yes. PrivateLink can provide private connectivity to a supported endpoint service outside the consumer's VPC, including a service in another AWS account or a supported SaaS provider.

However, it does **not** provide general connectivity to arbitrary networks or the public Internet.
