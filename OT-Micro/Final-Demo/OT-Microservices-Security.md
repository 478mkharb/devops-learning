# OT-Microservices — Security Aspect Interview Questions

## 1. How is OT-Microservices secured from the Internet?

**Answer:** The Application Load Balancer (ALB) is the controlled Internet-facing entry point. Backend EC2 instances are deployed in private subnets and do not have public IP addresses.

```text
Internet
   |
   | HTTPS :443
   v
ALB
   |
   | VPC private network
   v
Private Backend EC2
   |
   v
Private Database Subnet
```

The backend is accessible through the ALB when an API endpoint must be public, but the backend EC2 itself is not directly Internet-accessible.

---

## 2. If the ALB is public, doesn't that mean the backend APIs are exposed?

**Answer:** The API endpoint is intentionally exposed through the ALB because clients need to consume the API. However, the backend network endpoint is not directly exposed.

For example:

```text
https://api.example.com/api/v1/employee/search/all
        |
        v
      ALB :443
        |
        v
Employee API private-ip:8080
```

A client cannot directly access:

```text
http://10.0.2.25:8080/api/v1/employee/search/all
```

The distinction is between exposing an application endpoint through a controlled load balancer and exposing the backend EC2 directly to the Internet.

---

## 3. How does the ALB communicate with an EC2 in a private subnet?

**Answer:** The ALB and backend instances are in the same VPC. The VPC route table contains the local VPC route, allowing private subnet-to-private subnet communication.

Example:

```text
ALB:          10.0.1.50
Employee EC2: 10.0.2.25

10.0.0.0/16 -> local
```

The ALB forwards traffic to the target's private IP and application port. No Internet Gateway is required for ALB-to-backend communication.

---

## 4. How do Security Groups protect the backend?

**Answer:** The backend Security Group allows application traffic only from the ALB Security Group.

```text
Employee-API-SG

Inbound:
TCP 8080
Source: ALB-SG
```

It should NOT be:

```text
TCP 8080
Source: 0.0.0.0/0
```

The intended path is:

```text
Client
  |
  | 443
  v
ALB-SG
  |
  | 8080
  v
Employee-API-SG
```

---

## 5. Why use Security Group references instead of IP addresses?

**Answer:** Referencing the ALB Security Group provides dynamic and least-privilege access control.

If an Auto Scaling Group creates additional backend instances, they can all use the same Backend Security Group.

```text
             ALB-SG
                |
                v
        Backend-SG
        /    |    \
   EC2-1   EC2-2   EC2-3
```

There is no need to maintain individual backend IP addresses in the Security Group.

---

## 6. Can a user directly access the private backend IP?

**Answer:** No, not from the public Internet. For example:

```text
10.0.2.25:8080
```

is a private VPC address. The backend also has no public IP and its Security Group does not allow Internet-originated application traffic.

The user must access the public ALB endpoint.

---

## 7. Can users access `/api/v1/employee/search/all`?

**Answer:** Yes, if that endpoint is intentionally published through the ALB.

```text
Client
  |
  | HTTPS
  v
ALB
  |
  | Listener rule:
  | /api/v1/employee/*
  v
Employee Target Group
  |
  v
Employee API :8080
```

The API is externally accessible through the ALB, but the backend instance remains private.

---

## 8. What is the role of ALB listener rules?

**Answer:** Listener rules provide Layer-7 request routing based on conditions such as URL path or host header.

Example:

```text
/api/v1/employee/*
        -> Employee Target Group

/api/v1/attendance/*
        -> Attendance Target Group

/api/v1/salary/*
        -> Salary Target Group

/api/v1/notification/*
        -> Notification Target Group
```

This provides a controlled single ingress point for the APIs.

---

## 9. What is the difference between Route 53 and the ALB?

**Answer:** Route 53 provides DNS resolution. The ALB performs load balancing and application-level request routing.

```text
api.example.com
      |
      v
Route 53
      |
      v
ALB
      |
      v
Target Group
      |
      v
Private EC2
```

Route 53 answers: **Which address belongs to this DNS name?**

The ALB answers: **Which healthy backend target should receive this request?**

---

## 10. How does Route 53 Private Hosted Zone improve security?

**Answer:** Internal names such as:

```text
otms.employee.internal
otms.attendance.internal
otms.salary.internal
otms.postgresql.internal
otms.redis.internal
otms.scylladb.internal
```

can be resolved inside the private networking environment. They resolve to private addresses and do not make the services Internet-facing.

```text
Frontend
   |
   | otms.employee.internal
   v
Route 53 Private Hosted Zone
   |
   v
10.0.2.25
   |
   v
Employee API
```

---

## 11. Does Route 53 actually route network packets?

**Answer:** No. Route 53 performs DNS resolution.

```text
Route 53
    |
    | DNS resolution
    v
Private IP

Route Table
    |
    | network path
    v
Destination

Security Group
    |
    | authorization
    v
Allow / Deny
```

---

## 12. What happens if a private EC2 accidentally gets a public IP?

**Answer:** A public IP by itself does not automatically make the instance Internet-reachable. Internet reachability also depends on routing and security controls.

However, assigning a public IP to a private backend is bad practice because it increases the attack surface and can create unintended exposure if subnet routing and security rules permit Internet access.

Preferred design:

```text
Backend EC2
Private IP only
No public IP
Private subnet
```

---

## 13. What makes a subnet public?

**Answer:** A subnet is commonly considered public when its route table has a default route to an Internet Gateway.

```text
0.0.0.0/0 -> Internet Gateway
```

A private subnet typically has:

```text
0.0.0.0/0 -> NAT Gateway
```

for outbound Internet connectivity.

---

## 14. Why don't private backend instances need public IPs?

**Answer:** They receive inbound application traffic from the ALB over private VPC networking.

For outbound Internet access, they can use a NAT Gateway:

```text
Private EC2
    |
    v
NAT Gateway
    |
    v
Internet Gateway
    |
    v
Internet
```

The NAT Gateway provides outbound Internet connectivity without exposing the private EC2 directly to inbound Internet connections.

---

## 15. How are databases protected?

**Answer:** Databases should be placed in private/database subnets and protected with dedicated Security Groups.

Example:

```text
PostgreSQL-SG
Inbound:
TCP 5432
Source: Attendance-API-SG
```

Redis:

```text
Redis-SG
Inbound:
TCP 6379
Source: Application-SG
```

ScyllaDB:

```text
Scylla-SG
Inbound:
TCP 9042
Source: Salary-API-SG
```

The database should not accept these ports from:

```text
0.0.0.0/0
```

---

## 16. Explain the Security Group chain in OTMS.

**Answer:**

```text
Internet
   |
   | 443
   v
ALB-SG
   |
   | application ports
   v
Frontend-SG / Backend-SG
   |
   | database ports
   v
Database-SG
```

Each layer only permits the traffic required by the next layer.

---

## 17. Can the Internet directly access PostgreSQL, Redis, or ScyllaDB?

**Answer:** No, not in the intended architecture.

They should be private and have Security Groups that allow traffic only from the application Security Groups that require them.

```text
Internet
   X
   |
PostgreSQL :5432
```

while:

```text
Attendance API
      |
      | 5432
      v
PostgreSQL
```

is allowed.

---

## 18. How does the ALB improve security besides routing?

**Answer:** The ALB provides a centralized ingress point.

Security-related capabilities can include:

- HTTPS/TLS termination
- Listener rules
- Target health checks
- Security Group enforcement
- Centralized access logging
- Integration with AWS WAF
- Controlled target exposure

This avoids exposing every backend instance directly to the Internet.

---

## 19. What is defense in depth in OTMS?

**Answer:** Security is provided through multiple layers rather than a single control.

```text
Internet
   |
   v
Public ALB
   |
   | Security Group
   v
Private Application EC2
   |
   | Security Group
   v
Private Database
```

Additional controls can include:

- HTTPS/TLS
- AWS WAF
- IAM least privilege
- SSM instead of SSH
- Private subnets
- Security Groups
- Network ACLs where appropriate
- CloudWatch/Prometheus/Grafana monitoring
- Centralized logs with Grafana Alloy + Loki

---

## 20. Why use SSM instead of exposing SSH?

**Answer:** SSM allows administrative access without requiring the application EC2 to expose port 22 to the Internet.

```text
Administrator
      |
      v
AWS Systems Manager
      |
      v
SSM Agent
      |
      v
Private EC2
```

This reduces the need for public IPs and inbound SSH rules.

---

## 21. What if the reviewer says: "Your ALB is exposing backend services"?

**Best answer:**

> "The API endpoint is intentionally exposed through the ALB because clients need to consume it. What is not exposed is the backend network endpoint. The backend EC2 instances are private, have no public IPs, and their Security Groups allow application traffic only from the ALB Security Group. The ALB terminates the client connection and establishes a separate connection to the private target. Therefore the ALB is the controlled ingress point, while the backend infrastructure remains isolated from direct Internet access."

---

## 22. What if the reviewer asks: "Can I bypass the ALB?"

**Answer:**

> "Not from the public Internet. The backend has no public IP and the application port is not open to the Internet. The backend Security Group only permits traffic from the ALB Security Group."

```text
Internet
   |
   +----> ALB :443 ----> Backend :8080   ✓
   |
   +----> Backend :8080                  ✗
```

---

## 23. What if the reviewer says: "But anyone can call your API URL"?

**Answer:**

> "Yes, the public API endpoint can be called because that is required functionality. Security does not mean making the API unreachable. Security means controlling how it is reached and what can be reached. The public endpoint terminates at the ALB, while the backend target remains private and protected by Security Groups."

For stronger application-layer protection, AWS WAF, authentication/authorization, rate limiting, and API-specific controls can be added as appropriate.

---

## 24. What is the difference between network exposure and application exposure?

**Answer:**

**Application exposure:**

```text
/api/v1/employee/search/all
```

is available to a client through the ALB.

**Network exposure:**

```text
10.0.2.25:8080
```

is the private backend endpoint and is not directly exposed to the Internet.

This distinction is important when discussing ALB security.

---

## 25. Give the complete OTMS security flow.

**Answer:**

```text
                         INTERNET
                            |
                            | HTTPS :443
                            v
                    Public Route 53
                            |
                            v
                     +------------+
                     |    ALB     |
                     |   ALB-SG   |
                     +-----+------+
                           |
                    VPC Local Routing
                           |
             +-------------+-------------+
             v             v             v
        Frontend-SG   Employee-SG    Salary-SG
             |             |             |
             v             v             v
          NGINX        Employee API   Salary API
                           |
                    Private Route 53
                           |
             +-------------+-------------+
             v             v             v
        PostgreSQL       Redis        ScyllaDB
        Database-SG     Redis-SG      Scylla-SG
```

### Security principles

1. Single Internet ingress: ALB.
2. Private backend: no public IP.
3. Private databases: no Internet exposure.
4. Least-privilege Security Groups: source SG references instead of `0.0.0.0/0`.
5. Private DNS: Route 53 Private Hosted Zones for internal service discovery.
6. Controlled routing: ALB listener rules and target groups.
7. Outbound Internet: NAT Gateway when required.
8. Administration: SSM instead of exposing SSH.
9. Optional Layer-7 protection: AWS WAF.
10. Observability: centralized metrics and logs.

---

# Quick Reviewer Cheat Sheet

| Reviewer Question | Key Point |
|---|---|
| Is ALB public? | Yes, intentionally; it is the ingress point |
| Are backend EC2s public? | No |
| Do backend EC2s have public IPs? | No |
| How does ALB reach backend? | VPC private networking |
| What protects backend ports? | Backend Security Group |
| Backend SG source? | ALB Security Group |
| Can Internet directly reach backend? | No |
| Can users access APIs? | Yes, through ALB |
| Does Route 53 expose private services? | No |
| What does Route 53 do? | DNS resolution |
| What does the route table do? | Network path selection |
| What does SG do? | Traffic authorization |
| How do private EC2s reach Internet? | NAT Gateway, when required |
| How are databases protected? | Private subnet + dedicated DB SG |
| How is admin access handled? | AWS Systems Manager/SSM |
| Additional web protection? | AWS WAF, if implemented |

---

# One-Minute Final Answer

> "OT-Microservices follows a layered security model. The Internet-facing ALB is the single controlled ingress point. Backend EC2 instances run in private subnets without public IPs, and the ALB reaches them using private VPC networking. Each backend Security Group allows only the required application port from the ALB Security Group, so clients cannot directly connect to the backend instances. Route 53 Private Hosted Zones provide internal DNS for service-to-service communication without making those services public. Databases are isolated in private database subnets with dedicated Security Groups. For outbound connectivity, private instances use NAT Gateway or VPC endpoints as appropriate, and administrative access can use SSM rather than exposing SSH. Therefore, the API may be intentionally reachable through the ALB, but the underlying backend infrastructure is not directly Internet-accessible."
