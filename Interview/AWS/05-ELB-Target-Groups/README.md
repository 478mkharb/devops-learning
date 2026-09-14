# ELB & Target Groups Interview Notes

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is Elastic Load Balancing?](#2-what-is-elastic-load-balancing)
3. [Why Do We Need a Load Balancer?](#3-why-do-we-need-a-load-balancer)
4. [Types of AWS Load Balancers](#4-types-of-aws-load-balancers)
5. [Application Load Balancer](#5-application-load-balancer)
6. [Network Load Balancer](#6-network-load-balancer)
7. [Gateway Load Balancer](#7-gateway-load-balancer)
8. [Classic Load Balancer](#8-classic-load-balancer)
9. [ALB vs NLB vs GWLB](#9-alb-vs-nlb-vs-gwlb)
10. [ELB Architecture](#10-elb-architecture)
11. [Listeners](#11-listeners)
12. [ALB Listener Rules](#12-alb-listener-rules)
13. [Target Groups](#13-target-groups)
14. [Target Types](#14-target-types)
15. [Health Checks](#15-health-checks)
16. [Load-Balancing Algorithms](#16-load-balancing-algorithms)
17. [Session Affinity / Sticky Sessions](#17-session-affinity--sticky-sessions)
18. [NLB Source-IP Stickiness](#18-nlb-source-ip-stickiness)
19. [GWLB Flow Stickiness](#19-gwlb-flow-stickiness)
20. [Cross-Zone Load Balancing](#20-cross-zone-load-balancing)
21. [Deregistration Delay / Connection Draining](#21-deregistration-delay--connection-draining)
22. [Slow Start](#22-slow-start)
23. [Weighted Target Groups and Canary Deployments](#23-weighted-target-groups-and-canary-deployments)
24. [TLS Termination and Certificates](#24-tls-termination-and-certificates)
25. [Source IP Preservation](#25-source-ip-preservation)
26. [Proxy Protocol v2](#26-proxy-protocol-v2)
27. [ALB Authentication and AWS WAF](#27-alb-authentication-and-aws-waf)
28. [Access Logging and Monitoring](#28-access-logging-and-monitoring)
29. [Availability Zones and High Availability](#29-availability-zones-and-high-availability)
30. [Scaling and Quotas](#30-scaling-and-quotas)
31. [Security Groups and Network ACLs](#31-security-groups-and-network-acls)
32. [Common Failure Scenarios](#32-common-failure-scenarios)
33. [Common AWS CLI Commands](#33-common-aws-cli-commands)
34. [Frequently Asked Interview Questions](#34-frequently-asked-interview-questions)
35. [One-Line Interview Answers](#35-one-line-interview-answers)
36. [Official References](#36-official-references)

---

# 1. Introduction

**Elastic Load Balancing (ELB)** is AWS's managed load-balancing service. It distributes incoming traffic across registered targets and uses health checks so that traffic is normally sent only to healthy targets.

```text
Client
  |
  v
Load Balancer
  |
  +---- Target 1
  +---- Target 2
  +---- Target 3
```

ELB is commonly used for:

* High availability
* Horizontal scaling
* Rolling deployments
* Microservices routing
* TLS termination
* Health-based traffic distribution

---

# 2. What is Elastic Load Balancing?

Elastic Load Balancing distributes incoming traffic across healthy backend targets.

AWS provides:

* **Application Load Balancer (ALB)** — application-layer HTTP/HTTPS routing
* **Network Load Balancer (NLB)** — transport-layer, high-performance traffic handling
* **Gateway Load Balancer (GWLB)** — deployment and scaling of virtual network appliances
* **Classic Load Balancer (CLB)** — legacy/previous-generation option

The load balancer gives clients a stable entry point while backend targets can be added, removed, or replaced.

---

# 3. Why Do We Need a Load Balancer?

Without a load balancer:

```text
Client
  |
  v
Single Server
```

Problems:

* Single point of failure
* Limited capacity
* Difficult horizontal scaling
* Maintenance can cause downtime

With a load balancer:

```text
              Load Balancer
              /     |      \
             /      |       \
          App-1    App-2    App-3
```

The load balancer can stop sending traffic to an unhealthy target while continuing to use healthy targets.

---

# 4. Types of AWS Load Balancers

| Load Balancer | Primary Layer | Main Use Case |
|---|---:|---|
| ALB | L7 | HTTP/HTTPS, APIs, microservices, application-aware routing |
| NLB | L4 | TCP/UDP/TLS, low latency, high throughput, static IP requirements |
| GWLB | L3 appliance model | Firewalls and other network appliances |
| CLB | Legacy | Older workloads |

---

# 5. Application Load Balancer

An **Application Load Balancer (ALB)** operates at **Layer 7** and understands HTTP/HTTPS.

It can route using information such as:

* Host header
* URL path
* HTTP method
* HTTP headers
* Query string
* Source IP

Example:

```text
                 ALB
                  |
       +----------+----------+
       |          |          |
    /api/*    /admin/*   /images/*
       |          |          |
    API TG     Admin TG    Web TG
```

ALB is normally the preferred choice when traffic is HTTP/HTTPS and application-aware routing is required.

---

# 6. Network Load Balancer

A **Network Load Balancer (NLB)** operates primarily at **Layer 4**.

It is designed for:

* TCP
* UDP
* TLS
* High throughput
* Low latency
* Static IP requirements
* Source-IP preservation scenarios

NLB creates a network interface in each enabled Availability Zone. For internet-facing designs, an Elastic IP can be associated with each subnet. [AWS NLB documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)

NLB is not selected when path-based or host-based HTTP routing is required; that is an ALB use case.

---

# 7. Gateway Load Balancer

A **Gateway Load Balancer (GWLB)** is designed to deploy and scale virtual network appliances such as:

* Firewalls
* Intrusion-prevention systems
* Deep-packet inspection appliances
* Other security appliances

GWLB uses **GENEVE** and port **6081** between the GWLB and appliance targets. AWS documents GWLB as operating at Layer 3 for its traffic-appliance model. [AWS GWLB documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/gateway-load-balancers.html)

Simplified:

```text
Traffic
  |
  v
GWLB Endpoint
  |
  v
  GWLB
  |
  v
Security Appliance Fleet
  |
  v
Destination
```

---

# 8. Classic Load Balancer

**Classic Load Balancer (CLB)** is the earlier generation of Elastic Load Balancing.

It provides basic load-balancing capabilities but does not provide the full feature set of ALB and NLB.

For new architectures, ALB, NLB, or GWLB is normally selected according to workload requirements.

CLB remains relevant mainly for legacy systems and migration work.

---

# 9. ALB vs NLB vs GWLB

| Feature | ALB | NLB | GWLB |
|---|---|---|---|
| Primary model | L7 | L4 | L3 appliance model |
| HTTP-aware routing | Yes | No | No |
| Host/path routing | Yes | No | No |
| TCP/UDP focus | No | Yes | Appliance traffic |
| TLS termination | Yes | Yes | Appliance-oriented |
| Static IP requirement | Not the primary reason | Strong fit | Networking model |
| Security appliances | No | No | Yes |
| Microservice routing | Excellent | Limited | No |

### Choose ALB

```text
HTTP/HTTPS
+
Host/path/header rules
+
Application-aware routing
```

### Choose NLB

```text
TCP/UDP/TLS
+
High throughput / low latency
+
Static IP
+
Transport-layer load balancing
```

### Choose GWLB

```text
Need to insert and scale
firewalls/security appliances
```

---

# 10. ELB Architecture

The key components are:

```text
Load Balancer
    |
    +---- Listener
    |       |
    |       +---- Listener Rules
    |
    +---- Target Group
            |
            +---- Target 1
            +---- Target 2
            +---- Target 3
```

## Load Balancer

The client-facing entry point.

## Listener

Accepts connections on a protocol and port.

## Listener Rule

Determines what happens to matching requests, primarily for ALB.

## Target Group

A logical collection of backend targets.

## Target

The backend destination that receives traffic.

---

# 11. Listeners

A **listener** accepts connection requests on a configured protocol and port.

Examples:

```text
HTTP  : 80
HTTPS : 443
TCP   : 80
TLS   : 443
UDP   : 53
```

For an ALB:

```text
Client
  |
  v
HTTPS :443
  |
  v
Listener
  |
  v
Rules
  |
  v
Target Group
```

An HTTPS listener requires a server certificate and a TLS security policy. [AWS HTTPS listener documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html)

---

# 12. ALB Listener Rules

An ALB listener rule contains:

* Priority
* Conditions
* Actions
* Optional transforms

Rules are evaluated from the lowest priority number to the highest.

Common actions:

* Forward to target group
* Redirect
* Fixed response

Example:

```text
HTTPS :443
    |
    +---- Host = api.example.com
    |          -> API Target Group
    |
    +---- Path = /admin/*
    |          -> Admin Target Group
    |
    +---- Default
               -> Web Target Group
```

AWS documents host-header, path-pattern, HTTP-method, HTTP-header, query-string, and source-IP conditions for ALB listener rules. [AWS ALB listener rules](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-rules.html)

---

# 13. Target Groups

A **target group** is a logical collection of backend targets.

It defines or contains settings for:

* Target type
* Protocol
* Port
* Health checks
* Routing algorithm
* Stickiness
* Deregistration delay
* Slow start
* Cross-zone behavior, where applicable

Example:

```text
ALB
 |
 +---- /api  -> api-tg
 |              +-- EC2-1
 |              +-- EC2-2
 |
 +---- /web  -> web-tg
                +-- EC2-3
                +-- EC2-4
```

ALB and NLB target groups use target-group settings to determine how traffic is forwarded and how targets are health checked. [AWS ALB target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) [AWS NLB target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html)

---

# 14. Target Types

## ALB

Common ALB target types include:

* `instance`
* `ip`
* `lambda`

## NLB

Common NLB target types include:

* `instance`
* `ip`
* `alb`

The target type affects how targets are registered and can also affect client-IP behavior. [AWS NLB target types](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html)

### Why use IP targets?

Useful when the backend is reached by IP rather than an EC2 instance ID, such as:

* Containers
* Private IP endpoints
* Pod/service endpoints
* Hybrid environments

---

# 15. Health Checks

A health check periodically tests whether a registered target is healthy.

Common settings include:

* Protocol
* Port
* HTTP/HTTPS path
* Timeout
* Interval
* Healthy threshold
* Unhealthy threshold
* Success criteria

Example:

```text
Target Group
   |
   +-- EC2-1 -> healthy
   +-- EC2-2 -> healthy
   +-- EC2-3 -> unhealthy
```

The load balancer normally sends new traffic only to targets that pass health checks. Health checks are configured per target group. [AWS ALB target-group health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

---

# 16. Load-Balancing Algorithms

For ALB target groups, AWS documents these routing algorithms:

## Round Robin

```text
Request 1 -> A
Request 2 -> B
Request 3 -> C
Request 4 -> A
```

Useful when targets have similar capacity and requests have similar cost.

## Least Outstanding Requests

Routes requests toward targets with fewer in-progress requests.

Useful when request processing times vary.

## Weighted Random

Selects targets randomly using a weighted algorithm. AWS documents Automatic Target Weights anomaly mitigation with this algorithm.

Important compatibility note: AWS documents that weighted-random cannot be combined with sticky sessions or slow start for ALB target groups. [AWS ALB routing algorithms](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html)

---

# 17. Session Affinity / Sticky Sessions

**Sticky sessions**, also called **session affinity**, bind a client's requests to the same target for a configured period.

Without stickiness:

```text
Client
  |
  +-- Request 1 -> EC2-1
  +-- Request 2 -> EC2-2
  +-- Request 3 -> EC2-3
```

With stickiness:

```text
Client
  |
  +-- Request 1 -> EC2-1
  +-- Request 2 -> EC2-1
  +-- Request 3 -> EC2-1
```

## Why Use Sticky Sessions?

Useful when an application incorrectly or intentionally keeps session state locally on an instance.

Example:

```text
User Session
     |
     v
  EC2-1 local memory
```

All requests need to return to EC2-1.

## Better Architecture: Externalize Session State

In a scalable application, prefer:

```text
Client
  |
  v
ALB
  |
  +---- EC2-1
  +---- EC2-2
  +---- EC2-3
            |
            v
      Shared Session Store
```

This can reduce dependence on stickiness and allows better load distribution.

## ALB Stickiness Types

ALB supports:

```text
1. Duration-based cookie
2. Application-based cookie
```

ALB stickiness is configured at target-group level. AWS documents the `AWSALB`-based duration cookie and application-cookie mode. [AWS ALB sticky sessions](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html)

### Duration-Based Stickiness

The load balancer generates the stickiness cookie.

```text
First Request
    |
    v
ALB selects Target A
    |
    v
AWSALB cookie
    |
    v
Future requests
    |
    v
Target A
```

### Application-Based Stickiness

The application provides the cookie and ALB uses it as the basis for affinity.

### Important Drawback

Stickiness can create uneven distribution:

```text
Many clients
     |
     v
Same target
     |
     v
Hot target
```

Use affinity when the application needs it, not simply because it is available.

AWS documents that ALB sticky sessions require cross-zone load balancing to be enabled. [AWS ALB target-group attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html)

---

# 18. NLB Source-IP Stickiness

NLB supports target-group stickiness using `source_ip` for supported configurations.

```text
Client IP
   |
   v
NLB
   |
   v
Same target
```

This is different from ALB's cookie-based session affinity.

### NAT Caveat

Suppose:

```text
Client A --+
Client B --+--> NAT --> same source IP --> NLB
Client C --+
```

Source-IP stickiness can send those clients to the same backend target, creating uneven load.

[AWS NLB target-group attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/edit-target-group-attributes.html)

---

# 19. GWLB Flow Stickiness

GWLB must preserve a network flow on the same appliance when stateful inspection is involved.

The default flow-stickiness model is **5-tuple**:

```text
Source IP
Source Port
Destination IP
Destination Port
Protocol
```

GWLB can also use:

```text
3-tuple
2-tuple
```

depending on configuration.

This is different from ALB cookie affinity and NLB source-IP affinity.

[AWS GWLB flow stickiness](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/edit-target-group-attributes.html)

---

# 20. Cross-Zone Load Balancing

Cross-zone load balancing allows traffic to be distributed across healthy targets in enabled Availability Zones rather than limiting each load-balancer node to local-zone targets.

Example without useful cross-zone distribution:

```text
AZ-A
ALB Node -> A1, A2

AZ-B
ALB Node -> B1
```

If target counts differ significantly, distribution can become uneven.

Cross-zone behavior depends on the load balancer type and target-group configuration, so verify the applicable AWS configuration rather than assuming one universal default.

[AWS ALB target-group attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html) [AWS NLB target-group attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/edit-target-group-attributes.html)

---

# 21. Deregistration Delay / Connection Draining

When a target is removed:

```text
New requests  --X--> Target
Existing work -----> Target
```

Elastic Load Balancing allows existing in-flight work time to complete.

Typical target state transition:

```text
registered
    |
    v
 draining
    |
    v
 unused
```

For ALB target groups, the documented default deregistration delay is **300 seconds**, with a configurable range of **0–3600 seconds**. [AWS ALB deregistration delay](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html)

This is important during:

* Rolling deployment
* Auto Scaling scale-in
* Instance maintenance
* Blue/green replacement

---

# 22. Slow Start

Slow start gradually increases traffic to a newly registered target.

```text
New Target
   |
   v
Slow Start
   |
   +-- small share
   +-- larger share
   +-- full share
```

For ALB target groups, AWS documents a range of **30–900 seconds**, with `0` meaning disabled. [AWS ALB target-group attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html)

Useful for:

* JVM warm-up
* Cache warm-up
* Expensive application startup
* Newly launched Auto Scaling instances

---

# 23. Weighted Target Groups and Canary Deployments

ALB can forward to multiple target groups using relative weights.

Example:

```text
                ALB
                 |
          +------+------+
          |             |
         v1             v2
       weight 90     weight 10
```

Approximate distribution:

```text
90% -> v1
10% -> v2
```

Useful for:

* Canary deployment
* Blue/green deployment
* Gradual migration
* A/B testing

AWS documents weights from `0` to `999` for ALB forward actions. Importantly, weighted forwarding does **not** automatically fail over to another weighted target group just because one group is empty or all of its targets are unhealthy. [AWS ALB weighted forwarding](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/rule-action-types.html)

---

# 24. TLS Termination and Certificates

## TLS Termination

With TLS termination at the load balancer:

```text
Client
  |
  | HTTPS
  v
ALB / NLB
  |
  | HTTP or HTTPS/TLS
  v
Target
```

Benefits:

* Centralized certificates
* Reduced TLS work on targets
* Simpler application configuration

## ALB HTTPS

An ALB HTTPS listener requires:

* X.509 server certificate
* TLS security policy

ACM is commonly used for certificate management.

[ALB HTTPS listener](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html) [ALB certificates](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/https-listener-certificates.html)

## SNI

ALB can use Server Name Indication to serve multiple hostnames/certificates on the same secure listener.

```text
HTTPS :443
   |
   +-- api.example.com  -> Cert A
   +-- app.example.com  -> Cert B
```

## NLB TLS

NLB can terminate TLS using a TLS listener and a configured security policy.

[AWS NLB TLS security policies](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/describe-ssl-policies.html)

---

# 25. Source IP Preservation

Source-IP preservation means the backend receives the original client IP rather than only the load balancer's address.

Useful for:

* Audit logging
* IP allowlisting
* Security policy
* Rate limiting
* Application logs

For NLB, client-IP preservation depends on target type and protocol. AWS documents different behavior for instance and IP target groups. [AWS NLB client IP preservation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/edit-target-group-attributes.html)

For ALB HTTP applications, client address information is commonly exposed through forwarded headers such as:

```text
X-Forwarded-For
```

The application should only trust forwarded headers from trusted proxy/load-balancer paths.

---

# 26. Proxy Protocol v2

Proxy Protocol v2 passes connection metadata from a supported load balancer to the backend.

```text
Client
  |
  v
NLB
  |
  | Proxy Protocol v2
  v
Target
```

NLB exposes:

```text
proxy_protocol_v2.enabled
```

as a target-group attribute.

Do not enable it unless the backend understands Proxy Protocol v2; otherwise the backend may treat the proxy header as application data.

[AWS NLB target-group attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/edit-target-group-attributes.html)

---

# 27. ALB Authentication and AWS WAF

## ALB Authentication

ALB can authenticate users before forwarding requests when supported by its listener configuration.

This can centralize authentication at the load-balancer layer for suitable web applications.

ALB also supports token-validation features for supported configurations.

[AWS ALB listener rules](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-rules.html)

## AWS WAF

AWS WAF can protect an ALB by inspecting HTTP requests and allowing or blocking traffic based on WAF rules.

```text
Internet
   |
   v
AWS WAF
   |
   v
ALB
   |
   v
Target Group
```

Typical protections include:

* SQL injection rules
* XSS rules
* IP restrictions
* Rate-based rules
* Managed rule groups

[AWS ELB infrastructure security](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/infrastructure-security.html)

---

# 28. Access Logging and Monitoring

## ALB Access Logs

ALB access logs can record:

* Client IP
* Request path
* Request timing
* Status codes
* Target information

Traditional access logs can be stored in S3. AWS also provides newer integrations for ALB logs through CloudWatch Logs, Data Firehose, and S3. [AWS ALB access logs](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html)

## Connection Logs

Connection logs can provide:

* Client IP and port
* Listener port
* TLS protocol
* TLS cipher
* TLS handshake information
* Connection status
* Client certificate information

[AWS ALB connection logs](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-connection-logs.html)

## Health Check Logs

Health-check logs are useful when troubleshooting why a target is unhealthy. [AWS health-check logs](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-health-check-logs.html)

## CloudWatch Metrics

Common ALB metrics include:

```text
RequestCount
TargetResponseTime
HTTPCode_ELB_4XX_Count
HTTPCode_ELB_5XX_Count
HTTPCode_Target_4XX_Count
HTTPCode_Target_5XX_Count
HealthyHostCount
UnHealthyHostCount
ActiveConnectionCount
```

AWS publishes ELB metrics through CloudWatch. [AWS ALB CloudWatch metrics](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)

---

# 29. Availability Zones and High Availability

For high availability, deploy the load balancer across multiple Availability Zones.

```text
             Load Balancer
              /         \
            AZ-A       AZ-B
             |           |
          Targets      Targets
```

This helps prevent a single Availability Zone from becoming the only path to the application.

For production designs, also make sure backend target capacity exists in the enabled zones.

---

# 30. Scaling and Quotas

Elastic Load Balancing is a managed service that scales load-balancer capacity as traffic changes. [AWS NLB introduction](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)

Operational considerations include:

* Request volume
* New connections
* Active connections
* Processed bytes
* Number of rules
* Number of target groups
* Number of targets
* LCU requirements

## LCU

**LCU** means **Load Balancer Capacity Unit** and is part of ELB capacity/pricing measurement.

## Service Quotas

AWS publishes quotas for ALB, NLB, and GWLB, and some quotas are adjustable.

[ALB quotas](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-limits.html)

[NLB quotas](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-limits.html)

---

# 31. Security Groups and Network ACLs

## ALB

A common security model is:

```text
Internet
   |
   v
ALB Security Group
   |
   v
Target Security Group
```

Targets should generally accept application/health-check traffic only from the intended load-balancer/client path rather than being unnecessarily exposed to the internet.

## NLB

NLB networking differs because client-IP preservation and target type affect what source address the target sees.

Design the target security group based on the actual protocol, target type, and source-IP behavior.

## NACL

Network ACLs are subnet-level, stateless network filters.

They are separate from:

```text
Security Groups
ALB/NLB configuration
AWS WAF
```

[AWS ELB infrastructure security](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/infrastructure-security.html)

---

# 32. Common Failure Scenarios

## ALB Returns 503

Common first checks:

```text
Target Group
   |
   +-- Are there healthy targets?
   +-- Are health checks passing?
   +-- Is the listener rule correct?
```

Also verify:

* Health-check port
* Health-check path
* Security group
* NACL
* Application process

## Target is Unhealthy

Test the application directly from an appropriate network location:

```bash
curl http://<target-private-ip>:<port>/<health-path>
```

Check:

```text
Protocol
Port
Path
Expected response
Application binding
Security group
NACL
```

## Traffic Reaches Only One Target

Possible causes:

* Sticky sessions
* NLB source-IP stickiness
* Uneven target capacity
* Long-lived connections
* Cross-zone configuration
* NAT concentration

## Backend Sees Load Balancer IP

Check:

```text
NLB client-IP preservation
Proxy Protocol v2
ALB forwarded headers
Target type
```

## Deployment Drops Requests

Check:

```text
Deregistration delay
Graceful application shutdown
Connection draining
Auto Scaling lifecycle behavior
```

## 502 Bad Gateway

Common investigation areas:

* Backend connection failure
* Protocol mismatch
* TLS mismatch
* Invalid backend response
* Application failure

Confirm the exact failure using target health, load-balancer logs, application logs, and CloudWatch metrics.

---

# 33. Common AWS CLI Commands

## List Load Balancers

```bash
aws elbv2 describe-load-balancers
```

## List Target Groups

```bash
aws elbv2 describe-target-groups
```

## List Listeners

```bash
aws elbv2 describe-listeners \
  --load-balancer-arn <load-balancer-arn>
```

## List ALB Listener Rules

```bash
aws elbv2 describe-rules \
  --listener-arn <listener-arn>
```

## Describe Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

## Register Target

```bash
aws elbv2 register-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=i-0123456789abcdef0
```

## Deregister Target

```bash
aws elbv2 deregister-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=i-0123456789abcdef0
```

## Modify Target-Group Attributes

Example:

```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn <target-group-arn> \
  --attributes \
    Key=deregistration_delay.timeout_seconds,Value=60
```

Enable ALB stickiness:

```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn <target-group-arn> \
  --attributes \
    Key=stickiness.enabled,Value=true \
    Key=stickiness.lb_cookie.duration_seconds,Value=300
```

Enable NLB client-IP preservation where supported:

```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn <target-group-arn> \
  --attributes \
    Key=preserve_client_ip.enabled,Value=true
```

---

# 34. Frequently Asked Interview Questions

| Question | Answer |
|---|---|
| **What is Elastic Load Balancing?** | AWS service that distributes incoming traffic across healthy registered targets. |
| **What types of ELB exist?** | ALB, NLB, GWLB, and legacy Classic Load Balancer. |
| **What is ALB?** | A Layer 7 HTTP/HTTPS load balancer with application-aware routing. |
| **What is NLB?** | A Layer 4 load balancer for high-performance TCP/UDP/TLS and related transport workloads. |
| **What is GWLB?** | A load balancer used to deploy and scale virtual network appliances. |
| **Why choose ALB over NLB?** | When HTTP-aware routing such as host, path, header, or method matching is required. |
| **Why choose NLB over ALB?** | When Layer 4 performance, TCP/UDP, low latency, static IPs, or source-IP preservation are important. |
| **What is a listener?** | A configuration that accepts connections on a protocol and port and applies its default action/rules. |
| **What is an ALB listener rule?** | A priority-ordered set of conditions and actions used to route HTTP/HTTPS requests. |
| **What is a target group?** | A logical collection of backend targets with forwarding and health-check configuration. |
| **What is a target?** | The backend destination receiving traffic. |
| **What is a health check?** | A periodic test that determines whether a target is healthy enough to receive traffic. |
| **What happens when a target fails health checks?** | The load balancer stops routing new traffic to that target until it becomes healthy again. |
| **What is deregistration delay?** | The period used to allow in-flight work to complete when a target is being removed. |
| **What is connection draining?** | Preventing new traffic while allowing existing in-flight work to finish during deregistration. |
| **What is sticky session / session affinity?** | A mechanism that keeps a client's traffic associated with the same backend target. |
| **How does ALB stickiness work?** | ALB supports duration-based load-balancer cookies and application-based cookies. |
| **What is NLB source-IP stickiness?** | A target-selection method that uses the source IP as the stickiness key for supported NLB target groups. |
| **What is GWLB flow stickiness?** | Keeping a network flow on the same appliance using a configured 5-, 3-, or 2-tuple model. |
| **What is cross-zone load balancing?** | Distributing traffic across healthy targets in enabled Availability Zones instead of restricting each load-balancer node to local targets. |
| **What is slow start?** | Gradually increasing traffic to a newly registered target. |
| **What are weighted target groups?** | Forwarding traffic to multiple target groups using relative weights, useful for canary/blue-green deployment. |
| **Do weighted target groups automatically fail over if one group is unhealthy?** | No. AWS documents that weighted forwarding does not automatically fail over merely because another weighted group is empty or unhealthy. |
| **What is TLS termination?** | Decrypting client TLS at the load balancer and forwarding to the target using the configured backend protocol. |
| **What is source-IP preservation?** | Preserving the original client IP so the target can identify the client under supported configurations. |
| **What is Proxy Protocol v2?** | A protocol that passes connection metadata from a supported proxy/load balancer to the backend. |
| **What is SNI?** | Server Name Indication lets a TLS client indicate the hostname so the correct certificate can be selected. |
| **What is AWS WAF with ALB?** | A web application firewall layer that can allow/block requests using WAF rules. |
| **What is ALB access logging?** | Detailed logging of HTTP request activity for troubleshooting and traffic analysis. |
| **What is CloudWatch used for with ELB?** | Monitoring load-balancer and target metrics and creating alarms. |
| **Can one target belong to multiple target groups?** | Yes. A target can be registered with multiple target groups. |
| **Can one ALB serve multiple applications?** | Yes. Host/path/header/source-IP rules can route requests to different target groups. |
| **Can an NLB do path-based routing?** | No. Path/host routing is an ALB capability. |
| **Can ALB handle arbitrary TCP applications?** | No. ALB is designed for HTTP/HTTPS-aware traffic; use NLB for generic Layer 4 workloads. |
| **Why deploy load balancers across multiple AZs?** | To improve availability and avoid a single-AZ dependency. |
| **Can NLB have static IP addresses?** | Yes. NLB supports per-AZ static IP capability and can associate Elastic IPs for internet-facing designs. |
| **What is LCU?** | Load Balancer Capacity Unit, used in ELB capacity/pricing measurements. |
| **What is the difference between 502 and 503 at a high level?** | 502 commonly points to a bad/invalid upstream connection or response, while 503 commonly indicates no usable healthy target capacity; confirm with logs and metrics. |

---

# 35. One-Line Interview Answers

| Concept | One-Line Answer |
|---|---|
| **ELB** | AWS managed service for distributing traffic across healthy targets. |
| **ALB** | Layer 7 HTTP/HTTPS load balancer. |
| **NLB** | Layer 4 high-performance transport load balancer. |
| **GWLB** | Load balancer for scaling and inserting virtual network appliances. |
| **Listener** | Accepts connections on a configured protocol and port. |
| **Listener Rule** | Matches conditions and executes an action. |
| **Target Group** | Logical collection of backend targets. |
| **Target** | Backend destination receiving traffic. |
| **Health Check** | Determines whether a target should receive new traffic. |
| **Sticky Session** | Keeps a client associated with the same target. |
| **Session Affinity** | Another name for sticky-session behavior. |
| **Cross-Zone** | Allows distribution across healthy targets in enabled Availability Zones. |
| **Deregistration Delay** | Grace period for in-flight traffic during target removal. |
| **Slow Start** | Gradually increases traffic to new targets. |
| **Weighted Target Groups** | Sends different traffic proportions to different target groups. |
| **TLS Termination** | Load balancer decrypts frontend TLS traffic. |
| **Source-IP Preservation** | Backend receives the original client IP under supported configurations. |
| **Proxy Protocol v2** | Carries connection metadata to supported targets. |
| **SNI** | Allows TLS certificate selection by hostname. |
| **WAF** | Filters web requests using security rules. |
| **CloudWatch** | Provides ELB monitoring metrics and alarms. |
| **Access Logs** | Detailed request/connection logging for troubleshooting and analysis. |
| **LCU** | Load Balancer Capacity Unit used in ELB capacity/pricing measurements. |
| **Round Robin** | Sequentially distributes traffic across healthy targets. |
| **Least Outstanding Requests** | Prefers targets with fewer in-progress requests. |
| **Weighted Random** | Selects targets randomly using a weighted routing algorithm. |

---

# 36. Official References

The following AWS documentation should be used to verify current ELB features, defaults, quotas, and target-group behavior.

## Elastic Load Balancing

- https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html

## Application Load Balancer

- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-rules.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/rule-action-types.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/https-listener-certificates.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/describe-ssl-policies.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-monitoring.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html

## Network Load Balancer

- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/edit-target-group-attributes.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-listeners.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/describe-ssl-policies.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-limits.html

## Gateway Load Balancer

- https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/gateway-load-balancers.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/edit-target-group-attributes.html

## Security and Quotas

- https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/infrastructure-security.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-limits.html

> **Note:** AWS ELB capabilities, defaults, target types, quotas, and listener behavior can change. Always verify the exact behavior for the load balancer type, protocol, target type, and AWS Region used in production.
