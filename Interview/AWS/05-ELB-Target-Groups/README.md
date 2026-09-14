# Elastic Load Balancing (ELB) & Target Groups — Interview Notes

> **Scope:** This document covers only **Elastic Load Balancing (ELB)** and **Target Groups**.  
> Detailed topics such as listener rules, AWS WAF, Route 53, Security Groups, NACLs, Auto Scaling, TLS certificates, and application architecture are intentionally excluded.

---

## Table of Contents

1. [What is Elastic Load Balancing?](#1-what-is-elastic-load-balancing)
2. [Why Do We Need ELB?](#2-why-do-we-need-elb)
3. [Types of AWS Load Balancers](#3-types-of-aws-load-balancers)
4. [Application Load Balancer](#4-application-load-balancer)
5. [Network Load Balancer](#5-network-load-balancer)
6. [Gateway Load Balancer](#6-gateway-load-balancer)
7. [Classic Load Balancer](#7-classic-load-balancer)
8. [ELB Comparison](#8-elb-comparison)
9. [What is a Target Group?](#9-what-is-a-target-group)
10. [Relationship Between ELB and Target Groups](#10-relationship-between-elb-and-target-groups)
11. [Target Types](#11-target-types)
12. [Target Group Protocol and Port](#12-target-group-protocol-and-port)
13. [Health Checks](#13-health-checks)
14. [Target Health States](#14-target-health-states)
15. [Target Registration and Deregistration](#15-target-registration-and-deregistration)
16. [Deregistration Delay](#16-deregistration-delay)
17. [Load-Balancing Algorithms](#17-load-balancing-algorithms)
18. [Sticky Sessions](#18-sticky-sessions)
19. [Slow Start](#19-slow-start)
20. [Cross-Zone Load Balancing](#20-cross-zone-load-balancing)
21. [Weighted Target Groups](#21-weighted-target-groups)
22. [Target Group Attributes](#22-target-group-attributes)
23. [One Target in Multiple Target Groups](#23-one-target-in-multiple-target-groups)
24. [Common ELB and Target Group Failure Scenarios](#24-common-elb-and-target-group-failure-scenarios)
25. [Common AWS CLI Commands](#25-common-aws-cli-commands)
26. [Frequently Asked Interview Questions](#26-frequently-asked-interview-questions)
27. [One-Line Revision](#27-one-line-revision)

---

# 1. What is Elastic Load Balancing?

**Elastic Load Balancing (ELB)** is an AWS managed service that distributes incoming traffic across registered backend targets.

Targets may include:

- EC2 instances
- Private IP addresses
- Containers
- Lambda functions, where supported by the load balancer
- Another Application Load Balancer, where supported by NLB target groups

ELB can use target health information to avoid sending new traffic to unhealthy targets.

### Basic Architecture

```text
                    Clients
                       |
                       v
                Elastic Load Balancer
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Target 1  Target 2  Target 3
```

### Main Benefits

- Distributes traffic across multiple targets
- Improves application availability
- Supports horizontal scaling
- Removes unhealthy targets from normal traffic distribution
- Provides a stable entry point for clients
- Supports different traffic types depending on the load balancer type

---

# 2. Why Do We Need ELB?

Without a load balancer:

```text
Client
  |
  v
Single Server
```

Possible problems:

- The server may become overloaded.
- The server may become a single point of failure.
- Scaling requires changing the client-facing endpoint.
- Maintenance can interrupt traffic.
- Traffic cannot be distributed across multiple backend servers.

With ELB:

```text
                 ELB
                  |
        +---------+---------+
        |         |         |
      Server A  Server B  Server C
```

ELB distributes incoming traffic among available targets according to the load balancer and target-group configuration.

---

# 3. Types of AWS Load Balancers

| Load Balancer | Main Layer / Model | Main Use Case |
|---|---|---|
| **Application Load Balancer (ALB)** | Layer 7 | HTTP/HTTPS applications and application-aware routing |
| **Network Load Balancer (NLB)** | Layer 4 | TCP, UDP, TLS, high throughput, and low latency |
| **Gateway Load Balancer (GWLB)** | Network appliance model | Firewalls and other virtual network appliances |
| **Classic Load Balancer (CLB)** | Legacy generation | Older workloads |

### Important Interview Point

- **ALB** is generally selected for HTTP/HTTPS applications.
- **NLB** is generally selected for transport-layer traffic such as TCP, UDP, or TLS.
- **GWLB** is selected for inserting and scaling virtual network appliances.
- **CLB** is a legacy option and is normally not selected for new designs.

---

# 4. Application Load Balancer

An **Application Load Balancer (ALB)** operates at **Layer 7** and understands HTTP/HTTPS requests.

ALB is suitable for:

- Web applications
- REST APIs
- Microservices
- Host-based application routing
- Path-based application routing
- HTTP-header and method-based routing

### Common ALB Target Types

- `instance`
- `ip`
- `lambda`

### ALB Traffic Flow

```text
Client
  |
  v
ALB
  |
  v
Target Group
  |
  +---- EC2 instance
  +---- EC2 instance
  +---- IP target
```

### Important Point

An ALB is designed for HTTP/HTTPS-aware traffic. It is not the normal choice for arbitrary TCP or UDP applications.

---

# 5. Network Load Balancer

A **Network Load Balancer (NLB)** operates primarily at **Layer 4**.

NLB supports:

- TCP
- UDP
- TLS
- High-throughput workloads
- Low-latency workloads
- Static IP use cases
- Source-IP preservation scenarios, depending on configuration

### Common NLB Target Types

- `instance`
- `ip`
- `alb`

### NLB Traffic Flow

```text
Client
  |
  v
NLB
  |
  v
Target Group
  |
  +---- EC2 instance
  +---- Private IP
  +---- ALB, where supported
```

### Important Point

NLB does not provide ALB-style HTTP path-based or host-based routing.

---

# 6. Gateway Load Balancer

A **Gateway Load Balancer (GWLB)** is designed to deploy, scale, and integrate virtual network appliances.

Common appliance examples:

- Firewalls
- Intrusion-prevention systems
- Deep-packet inspection appliances
- Security inspection appliances

GWLB uses the GENEVE protocol and port `6081` between the GWLB and appliance targets.

### Simplified Flow

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
Virtual Network Appliance Fleet
```

### Important Point

GWLB is not a normal HTTP application load balancer. Its purpose is to work with network appliances.

---

# 7. Classic Load Balancer

**Classic Load Balancer (CLB)** is the earlier generation of Elastic Load Balancing.

It provides basic load-balancing capabilities but does not provide the full feature set of ALB and NLB.

### Interview Answer

> Classic Load Balancer is a legacy AWS load balancer. For new workloads, ALB, NLB, or GWLB is normally selected based on the traffic and architecture requirements.

---

# 8. ELB Comparison

| Feature | ALB | NLB | GWLB |
|---|---|---|---|
| Main model | Layer 7 | Layer 4 | Network appliance model |
| HTTP-aware | Yes | No | No |
| Host/path routing | Yes | No | No |
| TCP support | Not its primary use | Yes | Appliance traffic |
| UDP support | No | Yes | Appliance traffic |
| TLS handling | HTTPS listener support | TLS listener support | Appliance-oriented |
| Typical target types | Instance, IP, Lambda | Instance, IP, ALB | Appliance targets |
| Main use case | Web apps and APIs | Transport traffic | Security/network appliances |

### Selection Rule

```text
HTTP/HTTPS application
        |
        v
       ALB

TCP/UDP/TLS or static-IP requirement
        |
        v
       NLB

Firewall or network inspection appliance
        |
        v
      GWLB
```

---

# 9. What is a Target Group?

A **target group** is a logical collection of registered backend targets.

A target group is used by a load balancer to determine:

- Which targets can receive traffic
- The protocol used to forward traffic
- The port used to forward traffic
- How target health is checked
- Which target-selection behavior is used
- Whether features such as stickiness, slow start, or deregistration delay are enabled

### Example

```text
Target Group: api-tg
        |
        +---- EC2-1
        +---- EC2-2
        +---- EC2-3
```

Another target group may contain a different application:

```text
Target Group: web-tg
        |
        +---- EC2-4
        +---- EC2-5
```

### Important Point

A target group is not the same as a load balancer.

- **Load balancer:** Receives client traffic.
- **Target group:** Organizes backend targets and defines forwarding/health-check behavior.
- **Target:** Backend destination that receives traffic.

---

# 10. Relationship Between ELB and Target Groups

The general relationship is:

```text
Load Balancer
      |
      v
Target Group
      |
      v
Registered Targets
```

For an ALB, listener configuration determines which target group receives a request.

For an NLB, listener forwarding configuration determines which target group receives traffic.

### Example

```text
                  ALB
                   |
          +--------+--------+
          |                 |
       api-tg            web-tg
          |                 |
       EC2-1             EC2-3
       EC2-2             EC2-4
```

### Interview Answer

> A load balancer is the traffic entry point, while a target group is the logical collection of backend targets to which the load balancer forwards traffic.

---

# 11. Target Types

## 11.1 Instance Targets

The target is registered using an EC2 instance ID.

```text
Target Type: instance
Target: i-0123456789abcdef0
```

Useful when the backend is an EC2 instance.

## 11.2 IP Targets

The target is registered using an IP address.

```text
Target Type: ip
Target: 10.0.2.15
```

Useful for:

- Container IPs
- Private IP endpoints
- Pod or service endpoints
- Hybrid environments
- Backends not registered by EC2 instance ID

## 11.3 Lambda Targets

ALB can use Lambda functions as targets where supported.

```text
Client
  |
  v
ALB
  |
  v
Lambda Target
```

## 11.4 ALB Targets in NLB Target Groups

NLB supports an ALB target type where supported.

This allows an NLB to forward traffic to an ALB.

### Important Interview Point

Target types depend on the load balancer and target-group configuration. Do not assume every target type is supported by every load balancer.

---

# 12. Target Group Protocol and Port

A target group defines the protocol and port used to communicate with registered targets.

### Example

```text
Target Group
  Protocol: HTTP
  Port: 8080
```

The load balancer forwards traffic to the target on the configured target-group protocol and port, unless a registered target uses an explicitly configured port where supported.

### Common Examples

| Application | Target Group Protocol | Target Group Port |
|---|---|---:|
| HTTP application | HTTP | 80 |
| HTTPS application | HTTPS | 443 |
| Spring Boot application | HTTP | 8080 |
| Python Flask application | HTTP | 5000 |
| Node.js application | HTTP | 3000 |

### Important Point

The frontend listener port and backend target-group port do not have to be the same.

Example:

```text
Client
  |
  | HTTPS :443
  v
Load Balancer
  |
  | HTTP :8080
  v
Target Group
  |
  v
Application
```

---

# 13. Health Checks

A **health check** periodically tests whether a registered target is healthy.

Health checks are configured at the target-group level.

### Common Health-Check Settings

- Health-check protocol
- Health-check port
- Health-check path for HTTP/HTTPS
- Timeout
- Interval
- Healthy threshold
- Unhealthy threshold
- Success criteria or matcher

### Example

```text
Target Group
      |
      +---- EC2-1 -> healthy
      +---- EC2-2 -> healthy
      +---- EC2-3 -> unhealthy
```

The load balancer normally sends new traffic only to targets considered healthy.

### Example HTTP Health Check

```text
Protocol: HTTP
Port: traffic-port
Path: /health
Expected response: 200
```

### Health-Check Troubleshooting

Check:

1. Is the application running?
2. Is the application listening on the expected port?
3. Is the health-check path correct?
4. Is the health-check protocol correct?
5. Does the application return the expected status code?
6. Can the load balancer reach the target?
7. Is the target registered in the correct target group?

### Interview Answer

> A target-group health check periodically verifies target availability. If a target fails health checks, the load balancer stops sending normal new traffic to it until it becomes healthy again.

---

# 14. Target Health States

Common target health states include:

| State | Meaning |
|---|---|
| `initial` | The target is being registered or has not completed health checks |
| `healthy` | The target is passing health checks |
| `unhealthy` | The target is failing health checks |
| `draining` | The target is being deregistered and existing work may continue |
| `unused` | The target is not currently being used by the load balancer |
| `unavailable` | Health status cannot currently be determined |

### Typical Lifecycle

```text
initial
   |
   v
healthy
   |
   v
unhealthy
   |
   v
healthy
```

During deregistration:

```text
healthy
   |
   v
draining
   |
   v
unused
```

---

# 15. Target Registration and Deregistration

## Registering a Target

Registration adds a backend target to a target group.

```bash
aws elbv2 register-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=i-0123456789abcdef0
```

## Deregistering a Target

Deregistration removes a target from normal new-traffic distribution.

```bash
aws elbv2 deregister-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=i-0123456789abcdef0
```

### Why Deregister a Target?

- Maintenance
- Rolling deployment
- Instance replacement
- Application troubleshooting
- Removing an unhealthy or retired backend

---

# 16. Deregistration Delay

**Deregistration delay** is the time allowed for existing in-flight work to complete after a target starts deregistration.

### Behavior

```text
New requests
     |
     X  No new traffic to draining target

Existing requests
     |
     v
Allowed to complete during draining
```

For ALB target groups, the commonly documented default is **300 seconds**, with a configurable range of **0–3600 seconds**.

### Example

```text
Target becomes draining
        |
        v
Existing requests continue
        |
        v
Deregistration delay expires
        |
        v
Target becomes unused
```

### Interview Answer

> Deregistration delay, also called connection draining, allows existing requests to finish gracefully before a target is completely removed from service.

---

# 17. Load-Balancing Algorithms

For ALB target groups, AWS documents these routing algorithms:

## 17.1 Round Robin

Requests are distributed sequentially across eligible targets.

```text
Request 1 -> Target A
Request 2 -> Target B
Request 3 -> Target C
Request 4 -> Target A
```

Useful when targets have similar capacity and requests have similar processing costs.

## 17.2 Least Outstanding Requests

Traffic is directed toward targets with fewer in-progress requests.

Useful when:

- Request processing times vary
- Some requests are long-running
- Target workload is uneven

## 17.3 Weighted Random

Targets are selected using a weighted random algorithm.

AWS also documents Automatic Target Weights anomaly mitigation with this algorithm.

### Important Compatibility Point

For ALB target groups, weighted-random routing cannot be combined with sticky sessions or slow start.

---

# 18. Sticky Sessions

**Sticky sessions**, also called **session affinity**, keep a client's requests associated with the same target for a configured period.

### Without Stickiness

```text
Client
  |
  +---- Request 1 -> Target A
  +---- Request 2 -> Target B
  +---- Request 3 -> Target C
```

### With Stickiness

```text
Client
  |
  +---- Request 1 -> Target A
  +---- Request 2 -> Target A
  +---- Request 3 -> Target A
```

## ALB Stickiness Types

ALB supports:

1. Duration-based cookie stickiness
2. Application-based cookie stickiness

### Duration-Based Cookie

The load balancer generates the cookie.

### Application-Based Cookie

The application provides the cookie, and the load balancer uses it for affinity.

### Drawback

Stickiness may create an uneven distribution:

```text
Many clients
     |
     v
Same target
     |
     v
Hot target
```

Use stickiness only when the application needs session affinity.

### Important Point

For ALB, stickiness is configured at target-group level.

---

# 19. Slow Start

**Slow start** gradually increases traffic to a newly registered target.

### Example

```text
New Target
    |
    v
Slow Start
    |
    +---- Small traffic share
    +---- Larger traffic share
    +---- Normal traffic share
```

Useful for:

- JVM warm-up
- Cache warm-up
- Expensive application startup
- Newly launched application instances

For ALB target groups, AWS documents a slow-start range of **30–900 seconds**, with `0` meaning disabled.

### Important Point

Slow start gives a new target time to warm up instead of immediately receiving its full share of traffic.

---

# 20. Cross-Zone Load Balancing

Cross-zone load balancing allows traffic to be distributed across healthy targets in enabled Availability Zones rather than restricting each load-balancer node to targets in only its local Availability Zone.

### Example

```text
Without effective cross-zone distribution:

AZ-A
  Load Balancer Node -> A1, A2

AZ-B
  Load Balancer Node -> B1
```

If target counts differ between zones, traffic may become uneven.

### Important Point

Cross-zone behavior depends on the load balancer type and configuration. Verify the exact behavior for ALB, NLB, or GWLB rather than assuming one universal default.

---

# 21. Weighted Target Groups

ALB can forward traffic to multiple target groups using relative weights.

### Example

```text
                  ALB
                   |
             +-----+-----+
             |           |
           v1            v2
        Weight 90     Weight 10
```

Approximate distribution:

```text
90% -> v1 target group
10% -> v2 target group
```

Useful for:

- Canary deployments
- Blue/green deployments
- Gradual migration
- A/B testing

### Important Point

Weighted forwarding does not automatically fail over to another weighted target group merely because one target group is empty or all its targets are unhealthy.

---

# 22. Target Group Attributes

Depending on the load balancer and target-group type, attributes may include:

- Deregistration delay
- Stickiness
- Slow start
- Load-balancing algorithm
- Client-IP preservation, where supported
- Proxy Protocol v2, where supported
- Cross-zone behavior, where applicable

### Important Point

Not every attribute is supported by every load balancer or target type.

Always verify:

1. Load balancer type
2. Target type
3. Protocol
4. Target-group type
5. AWS Region
6. Current AWS documentation

---

# 23. One Target in Multiple Target Groups

**Yes. A target can belong to multiple target groups**, subject to AWS service limits and configuration requirements.

### Example

```text
                 EC2 Instance
                 /           \
                v             v
             api-tg         admin-tg
```

This can be useful when:

- The same instance serves multiple applications
- Different load balancers use the same backend
- Different target groups use different health-check settings
- Different forwarding configurations are required

### Important Point

Each target-group registration can have its own port and target-group behavior where supported.

---

# 24. Common ELB and Target Group Failure Scenarios

## 24.1 Load Balancer Returns 503

Common checks:

- Does the target group contain registered targets?
- Are any targets healthy?
- Are health checks passing?
- Is the target-group port correct?
- Is the application running?
- Is the target registered in the correct target group?

## 24.2 Target is Unhealthy

Check:

```text
Target Group
    |
    +-- Protocol
    +-- Port
    +-- Health-check path
    +-- Expected response
    +-- Application process
    +-- Network reachability
```

Test the application from an appropriate network location:

```bash
curl http://<target-private-ip>:<port>/<health-path>
```

## 24.3 Traffic Reaches Only One Target

Possible causes:

- Sticky sessions
- Uneven target capacity
- Long-lived connections
- Cross-zone configuration
- Target health differences
- NLB source-IP stickiness, where configured
- NAT concentration, where source-IP stickiness is used

## 24.4 Deployment Drops Requests

Check:

- Deregistration delay
- Graceful application shutdown
- Target draining state
- Application termination behavior
- Whether the target was deregistered before shutdown

## 24.5 Target Group Has No Healthy Targets

Check:

1. Target registration
2. Target health state
3. Health-check protocol
4. Health-check port
5. Health-check path
6. Expected response code
7. Application binding address
8. Network reachability
9. Target-group configuration

---

# 25. Common AWS CLI Commands

## List Load Balancers

```bash
aws elbv2 describe-load-balancers
```

## List Target Groups

```bash
aws elbv2 describe-target-groups
```

## Describe a Target Group

```bash
aws elbv2 describe-target-groups \
  --target-group-arns <target-group-arn>
```

## Describe Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

## Register a Target

```bash
aws elbv2 register-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=i-0123456789abcdef0
```

## Register a Target on a Specific Port

```bash
aws elbv2 register-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=i-0123456789abcdef0,Port=8080
```

## Deregister a Target

```bash
aws elbv2 deregister-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=i-0123456789abcdef0
```

## Modify Target-Group Attributes

```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn <target-group-arn> \
  --attributes \
    Key=deregistration_delay.timeout_seconds,Value=60
```

## Enable ALB Stickiness

```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn <target-group-arn> \
  --attributes \
    Key=stickiness.enabled,Value=true \
    Key=stickiness.lb_cookie.duration_seconds,Value=300
```

---

# 26. Frequently Asked Interview Questions

| Question | Answer |
|---|---|
| **What is ELB?** | AWS managed service that distributes traffic across registered backend targets. |
| **What types of ELB exist?** | ALB, NLB, GWLB, and legacy Classic Load Balancer. |
| **What is ALB?** | A Layer 7 load balancer for HTTP/HTTPS applications. |
| **What is NLB?** | A Layer 4 load balancer for TCP, UDP, TLS, high throughput, and low latency. |
| **What is GWLB?** | A load balancer for deploying and scaling virtual network appliances. |
| **What is a target group?** | A logical collection of backend targets with forwarding and health-check settings. |
| **What is a target?** | A backend destination that receives traffic from the load balancer. |
| **Can a target group contain EC2 instances?** | Yes, when the target type is `instance`. |
| **Can a target group contain IP addresses?** | Yes, when the target type is `ip` and supported by the load balancer. |
| **Can ALB use Lambda as a target?** | Yes, ALB supports Lambda targets. |
| **Can NLB use an ALB as a target?** | Yes, NLB supports the `alb` target type where supported. |
| **What is a health check?** | A periodic test used to determine whether a target is healthy. |
| **Where are health checks configured?** | At target-group level. |
| **What happens when a target becomes unhealthy?** | The load balancer normally stops sending new traffic to that target. |
| **What is deregistration delay?** | The grace period that allows existing work to finish while a target is removed. |
| **What is connection draining?** | Another term commonly used for graceful target deregistration. |
| **What is slow start?** | Gradually increasing traffic to a newly registered target. |
| **What is stickiness?** | Keeping a client's requests associated with the same target. |
| **Where is ALB stickiness configured?** | At target-group level. |
| **What is round robin?** | Sequentially distributing requests across eligible targets. |
| **What is least outstanding requests?** | Preferring targets with fewer in-progress requests. |
| **What is weighted random?** | Selecting targets randomly using a weighted algorithm. |
| **Can one target belong to multiple target groups?** | Yes, subject to service limits and configuration requirements. |
| **Can the frontend and backend ports differ?** | Yes. The listener port and target-group port can be different. |
| **What causes a target to become unhealthy?** | Incorrect protocol, port, path, response, application status, or network reachability. |
| **What is a 503 commonly associated with?** | No usable healthy target capacity, though logs and target health should confirm the cause. |
| **What is the difference between ELB and a target group?** | ELB receives and distributes traffic; a target group organizes backend targets and health-check behavior. |
| **What is weighted target-group forwarding?** | Sending different proportions of traffic to multiple target groups. |
| **Does weighted forwarding automatically fail over if one group is unhealthy?** | No, not merely because the group is empty or unhealthy. |
| **What is cross-zone load balancing?** | Distributing traffic across healthy targets in enabled Availability Zones, depending on configuration. |

---

# 27. One-Line Revision

| Concept | One-Line Answer |
|---|---|
| **ELB** | AWS managed service for distributing traffic across registered targets. |
| **ALB** | Layer 7 HTTP/HTTPS load balancer. |
| **NLB** | Layer 4 TCP/UDP/TLS load balancer. |
| **GWLB** | Load balancer for virtual network appliances. |
| **CLB** | Legacy load balancer generation. |
| **Target Group** | Logical collection of backend targets. |
| **Target** | Backend destination receiving traffic. |
| **Instance Target** | Target registered by EC2 instance ID. |
| **IP Target** | Target registered by IP address. |
| **Lambda Target** | Lambda function used as an ALB target. |
| **Health Check** | Determines whether a target should receive new traffic. |
| **Healthy** | Target is passing health checks. |
| **Unhealthy** | Target is failing health checks. |
| **Draining** | Target is being removed while existing work may finish. |
| **Deregistration Delay** | Grace period for existing traffic during target removal. |
| **Round Robin** | Sequential target selection. |
| **Least Outstanding Requests** | Prefers targets with fewer in-progress requests. |
| **Weighted Random** | Random target selection using weights. |
| **Sticky Session** | Keeps a client's requests on the same target. |
| **Slow Start** | Gradually increases traffic to a new target. |
| **Cross-Zone Load Balancing** | Distributes traffic across healthy targets in enabled zones. |
| **Weighted Target Groups** | Sends different traffic proportions to target groups. |

---

> **Final interview tip:**  
> Always distinguish these three terms:
>
> - **Load Balancer:** Receives client traffic.
> - **Target Group:** Organizes backend targets and defines forwarding/health-check behavior.
> - **Target:** The actual backend destination receiving traffic.
