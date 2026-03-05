# Traffic Flow: Internet → ALB → Private EC2 (Inside AWS VPC)

This architecture is a **standard secure production pattern** used in AWS where the **Application Load Balancer (ALB) is exposed to the internet** but **EC2 instances remain private**.

The goal is:

* Prevent direct internet access to EC2
* Provide load balancing
* Enable high availability and scaling

---

# 1. Architecture Overview

```
                Internet Users
                      │
                      ▼
                 Route53 DNS
                      │
                      ▼
             Internet Gateway (IGW)
                      │
                      ▼
        Public Subnet (AZ1 / AZ2)
                      │
                      ▼
           Application Load Balancer
                (Listener 80/443)
                      │
                      ▼
                 Target Group
                      │
           ┌──────────┴──────────┐
           ▼                     ▼
   Private Subnet AZ1     Private Subnet AZ2
           │                     │
           ▼                     ▼
      EC2 Instance 1        EC2 Instance 2
           │                     │
           └──────────┬──────────┘
                      ▼
                   Database

```

---

# 2. Step‑by‑Step Traffic Flow

## Step 1 — User Request

A user opens the website:

```
https://example.com
```

The browser performs a **DNS lookup**.

Route53 resolves the domain to the **ALB DNS name**.

Example:

```
example.com → my-alb-123456.us-east-1.elb.amazonaws.com
```

---

## Step 2 — Traffic Enters AWS via Internet Gateway

The ALB is deployed in **public subnets**.

A public subnet contains a route:

```
0.0.0.0/0 → Internet Gateway
```

Traffic path becomes:

```
User
 ↓
Internet
 ↓
Internet Gateway
 ↓
Public Subnet
 ↓
Application Load Balancer
```

---

## Step 3 — ALB Listener Receives Request

The **ALB Listener** waits for incoming requests.

Typical configuration:

```
HTTP  : 80
HTTPS : 443
```

The listener evaluates **listener rules**.

Example:

```
/api      → API Target Group
/images   → Image Target Group
/*        → Web Target Group
```

---

## Step 4 — Request Routed to Target Group

After rule evaluation, the ALB forwards the request to a **Target Group**.

Example:

```
web-target-group
```

Targets inside group:

```
EC2 Instance (Private Subnet AZ1)
EC2 Instance (Private Subnet AZ2)
```

ALB routing algorithm:

```
Least Outstanding Requests
```

This helps distribute traffic efficiently.

---

## Step 5 — Traffic Reaches Private EC2

The ALB communicates with EC2 **using private IP addresses inside the VPC**.

```
ALB (public subnet)
        │
        ▼
Private Subnet
        │
        ▼
EC2 instance
```

Important:

* EC2 instances **do not require public IPs**
* They are reachable internally via **VPC networking**

---

# 3. Security Group Flow

## ALB Security Group

Inbound rules:

```
Allow 80 / 443 from 0.0.0.0/0
```

Outbound rules:

```
Allow traffic to EC2 security group
```

---

## EC2 Security Group

Inbound rules:

```
Allow port 80 from ALB Security Group
```

This ensures:

* EC2 instances cannot be accessed directly from internet
* Only ALB can communicate with EC2

---

# 4. Response Flow

Response returns through the same path.

```
EC2 Instance
     ↓
Application Load Balancer
     ↓
Internet Gateway
     ↓
User Browser
```

Because **Security Groups are stateful**, return traffic is automatically allowed.

---

# 5. Installing Packages from Private EC2

Private EC2 instances cannot access the internet directly.

To allow outbound internet access:

```
Private EC2
     ↓
NAT Gateway (Public Subnet)
     ↓
Internet Gateway
     ↓
Internet
```

This allows:

* OS updates
* Package downloads
* API calls

while keeping instances private.

---

# 6. Full Production Architecture (Highly Available)

```
                     Internet
                        │
                        ▼
                     Route53
                        │
                        ▼
                Application Load Balancer
             (Public Subnets in 2 AZs)
                │                │
                ▼                ▼
        Private Subnet AZ1   Private Subnet AZ2
                │                │
                ▼                ▼
           EC2 Instance      EC2 Instance
                │                │
                └───────┬────────┘
                        ▼
                      Database

Additional components:

Private EC2 → NAT Gateway → Internet
Logs → CloudWatch
Scaling → Auto Scaling Group
```

---

# 7. Key Benefits of This Architecture

| Feature           | Benefit                                       |
| ----------------- | --------------------------------------------- |
| Security          | EC2 instances are not exposed to the internet |
| High Availability | ALB distributes traffic across multiple AZs   |
| Scalability       | Auto Scaling can add/remove EC2 instances     |
| Fault Tolerance   | Unhealthy instances are removed automatically |
| SSL Offloading    | HTTPS termination handled by ALB              |

---

# 8. Common Interview Follow‑Up Questions

1. Why should EC2 instances be placed in **private subnets**?
2. What happens if **target group health checks fail**?
3. Why must ALB be deployed in **multiple AZs**?
4. How does **Auto Scaling integrate with ALB**?
5. How would you troubleshoot **ALB returning 502 errors**?
