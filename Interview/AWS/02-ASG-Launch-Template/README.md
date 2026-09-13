# Auto Scaling Group & Launch Template

### Q1. What is a Launch Template?

**Answer:** A Launch Template is a reusable EC2 launch specification. It can define the AMI, instance type, IAM instance profile, networking and security groups, EBS mappings, user data, monitoring, and instance metadata options. It is versioned so ASGs and other launch mechanisms can use a controlled configuration.

---

### Q2. What information belongs in a Launch Template?

**Answer:** A Launch Template can contain the AMI, instance type, key pair if used, IAM instance profile, network interfaces and security groups, EBS volume mappings, user data, monitoring settings, and instance metadata options. It can therefore provide a complete, repeatable instance-launch definition.

---

### Q3. Why is a Launch Template preferred over a Launch Configuration?

**Answer:** A Launch Configuration is the older Auto Scaling launch specification. It defines settings used when launching instances, but it is less capable than Launch Templates and does not provide Launch Template versioning. Launch Templates are the preferred choice for new designs.

---

### Q4. What is Launch Template versioning?

**Answer:** Each Launch Template can have multiple numbered versions. A new version records a new launch configuration without changing earlier versions, which allows controlled rollouts and rollback to a known configuration.

---

### Q5. How does an ASG select a Launch Template version?

**Answer:** An ASG references a Launch Template together with a version. The version can be a specific numbered version or a special reference such as the default/latest version, depending on the ASG configuration. Using an explicit version gives the most predictable launches.

---

### Q6. What is an Auto Scaling Group?

**Answer:** An Auto Scaling Group (ASG) maintains a fleet of EC2 instances within configured minimum, desired, and maximum capacity. It can launch and terminate instances, replace unhealthy instances, and change capacity in response to scaling policies or schedules.

---

### Q7. Explain minimum, desired, and maximum capacity.

**Answer:** Minimum capacity is the lowest number of instances the ASG should maintain. Desired capacity is the current target number of instances. Maximum capacity is the upper limit to which the ASG can scale.

**Example:**

```text
Min     = 2
Desired = 4
Max     = 10
```

The ASG normally maintains 4 instances. It can scale out up to 10 and should not scale below 2.

---

### Q8. What is health-check replacement?

**Answer:** ASG health-check replacement means the group terminates an instance that is considered unhealthy and launches a replacement so the group can return to its desired capacity. The group can use EC2 status checks and, when configured, ELB health checks.

---

### Q9. How does an ASG replace an unhealthy instance?

**Answer:** When the ASG determines that an instance is unhealthy, it marks the instance for replacement, terminates it, and launches a replacement according to the group's Launch Template and capacity settings. With ELB health checks enabled, an instance can also be replaced when the load balancer reports it unhealthy.

---

### Q10. What is a warm pool?

**Answer:** An ASG warm pool keeps pre-initialized instances outside the active InService fleet so they can be brought into service faster during scale-out. It is especially useful when application initialization or bootstrapping takes significant time.

---

### Q11. What is target tracking scaling?

**Answer:** Target tracking scaling automatically adjusts ASG capacity to keep a selected metric near a target value, such as average CPU utilization or request count per target. It is generally the simplest policy when the goal is to maintain a stable metric target.

**Example:**

```text
Target CPU = 50%

CPU rises to 70% → ASG scales out
CPU falls to 30% → ASG can scale in
Goal             → Keep CPU around 50%
```

You specify the target rather than manually defining the number of instances to add for every metric value.

---

### Q12. What is step scaling?

**Answer:** Step scaling uses CloudWatch alarm breaches and different scaling adjustments for different ranges of metric deviation. For example, a severe CPU breach can add more instances than a small breach.

**Example:**

| CPU utilization | Scaling action |
|---|---:|
| 50%–60% | Add 1 instance |
| 60%–70% | Add 2 instances |
| Above 70% | Add 3 instances |

The important point is that **the size of the scaling action depends on the size of the metric breach**.

---

### Q13. What is simple scaling?

**Answer:** Simple scaling performs a single scaling adjustment when a CloudWatch alarm is breached and historically relies on a cooldown period before another simple scaling action. Target tracking and step scaling are generally preferred for modern ASGs.

**Example:**

```text
CPU > 70%
    ↓
CloudWatch alarm
    ↓
Add 2 instances
    ↓
Cooldown
```

Unlike step scaling, the same predefined adjustment is used rather than defining different adjustments for different breach ranges.

---

### Q14. What is scheduled scaling?

**Answer:** Scheduled scaling changes ASG capacity at predefined times. It is appropriate when demand is predictable, such as a workload that consistently needs more instances during business hours.

**Example:**

```text
08:00 → Desired capacity = 5
20:00 → Desired capacity = 2
```

This is useful when the traffic pattern is known in advance.

---

### Q15. What is predictive scaling?

**Answer:** Predictive scaling uses historical utilization patterns and forecasting to prepare EC2 capacity ahead of expected demand. It is useful for recurring traffic patterns and complements reactive scaling policies.

**Example:**

```text
Historical traffic
       ↓
AWS forecasting
       ↓
Expected traffic increase at 09:00
       ↓
Capacity prepared before demand arrives
```

The key difference from scheduled scaling is that predictive scaling uses **forecasted demand**, while scheduled scaling uses a **predefined schedule**.

---

### Q16. Compare target tracking, step, simple, scheduled, and predictive scaling.

**Answer:** Each policy answers a different scaling requirement.

| Policy | Trigger | Scaling decision | Example |
|---|---|---|---|
| **Target tracking** | Metric moves away from target | AWS automatically adjusts capacity toward the target | Keep CPU around 50% |
| **Step scaling** | CloudWatch alarm + size of metric breach | Different adjustments for different breach ranges | CPU 60% → +1, CPU 80% → +3 |
| **Simple scaling** | CloudWatch alarm | One predefined adjustment | CPU > 70% → +2 |
| **Scheduled scaling** | Predetermined time | Predefined capacity/adjustment | 08:00 → 5 instances |
| **Predictive scaling** | Forecasted future demand | Prepares capacity ahead of expected demand | Forecast traffic spike → scale before spike |

### Quick distinction

```text
Target Tracking → "Keep the metric at X"
Step Scaling    → "How far did the metric breach?"
Simple Scaling  → "Alarm fired → perform this action"
Scheduled       → "At this time → change capacity"
Predictive      → "Demand is forecasted → prepare capacity"
```

---

### Q17. Which policy is generally simplest for maintaining a target metric?

**Answer:** **Target tracking scaling** is generally the simplest choice when the requirement is to maintain a metric around a target.

For example:

```text
Target CPU = 50%
```

The ASG automatically adjusts capacity to keep the metric close to that target.

---

### Q18. Can an ASG use multiple scaling policies?

**Answer:** Yes. An ASG can have multiple scaling policies, for example target tracking for CPU utilization and another policy for request count. The policies operate within the group's minimum and maximum capacity limits, and scale-in behavior is coordinated to avoid unnecessarily aggressive reduction.

**Example:**

```text
                    ASG
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
   CPU Target Tracking    Request-based policy
       CPU = 50%          Requests/Target
          │                     │
          └──────────┬──────────┘
                     ↓
              ASG capacity
```

Multiple policies are useful when different signals provide important information about application load.

---

### Q19. What is a mixed instances policy?

**Answer:** A mixed instances policy allows an ASG to launch multiple EC2 instance types and combine purchasing options such as On-Demand and Spot. This improves capacity flexibility and can reduce cost.

**Example:**

```text
ASG
├── c7i.large
├── c7a.large
└── c6i.large

Purchasing:
├── On-Demand base capacity
└── Spot additional capacity
```

This reduces dependence on a single instance type or Spot capacity pool.

---

### Q20. How do On-Demand and Spot instances work together in an ASG?

**Answer:** A mixed instances policy lets an ASG combine On-Demand and Spot instances. You can specify a base amount of On-Demand capacity and use Spot for additional capacity, with allocation strategies selecting suitable Spot pools. This balances availability and cost.

**Example:**

```text
Desired capacity = 6
On-Demand base   = 2
Remaining        = 4 Spot

Possible fleet:
2 On-Demand + 4 Spot
```

The exact fleet can change as Spot capacity availability changes.

---

### Q21. What are termination policies?

**Answer:** ASG termination policies determine which instances are selected when the group needs to scale in. They can consider factors such as Availability Zone balance, launch-template version or configuration age, and instance age. The goal is to remove capacity while maintaining a balanced and predictable fleet.

**Example:**

If an ASG has:

```text
AZ-a → 4 instances
AZ-b → 2 instances
```

and needs to terminate one instance, the termination process considers Availability Zone balance before selecting an instance to remove.

---

### Q22. What is instance refresh?

**Answer:** Instance Refresh replaces existing ASG instances with instances launched from a new Launch Template configuration. It supports controlled rollouts using health thresholds, warm-up, checkpoints, and rollback-related controls.

**Example:**

```text
Old Launch Template v1
        ↓
ASG instances
  EC2-1 EC2-2 EC2-3
        ↓
Instance Refresh
        ↓
Launch Template v2
        ↓
New instances replace old instances gradually
```

This is commonly used for AMI updates, application configuration changes, or security patches.

---

### Q23. What are lifecycle hooks?

**Answer:** A lifecycle hook pauses an EC2 instance during an ASG launch or termination transition. This gives time for custom actions such as bootstrapping, registration, connection draining, log collection, or cleanup before the instance continues to InService or terminates.

**Example:**

```text
Launch
  ↓
Pending:Wait
  ↓
Install/configure application
  ↓
Complete lifecycle action
  ↓
InService
```

For termination:

```text
InService
  ↓
Terminating:Wait
  ↓
Drain connections / collect logs
  ↓
Complete lifecycle action
  ↓
Terminated
```

---

### Q24. How can lifecycle hooks delay termination or launch completion?

**Answer:** During launch, a lifecycle hook can keep an instance in a pending lifecycle state while initialization or registration completes. During termination, it can keep the instance in a terminating state while connections are drained or cleanup runs. The lifecycle action must eventually be completed or it will time out according to the hook configuration.

---

### Q25. What is a cooldown period?

**Answer:** A scaling cooldown is a period intended to allow a previous scaling action to take effect before another simple scaling action occurs. Modern target tracking uses instance warmup to help prevent new instances from skewing scaling decisions.

**Important distinction:**

| Concept | Purpose |
|---|---|
| **Cooldown** | Historically associated with simple scaling; prevents another simple scaling action immediately after a previous one |
| **Instance warmup** | Gives newly launched instances time to initialize before their metrics are fully considered for scaling decisions |

---

### Q26. What is default instance warmup?

**Answer:** Default instance warmup is the ASG setting that specifies how long a newly launched instance is considered to be warming up before its metrics are fully considered for scaling decisions.

The value is **configured for the ASG**; it should not be confused with the cooldown period. The effective warm-up behavior can also be influenced by scaling-policy and instance-refresh settings.

---

### Q27. What is the difference between cooldown and instance warmup?

**Answer:** They solve different problems.

| Feature | Cooldown | Instance warmup |
|---|---|---|
| Main purpose | Prevent rapid repeated simple scaling actions | Allow a newly launched instance to initialize |
| Primarily associated with | Simple scaling | Target tracking and other modern ASG operations |
| Applies to | Scaling action timing | Newly launched instances |
| Goal | Let a previous scaling action settle | Prevent incomplete/new-instance metrics from misleading scaling decisions |

---

### Q28. Give a practical example of choosing the right ASG scaling policy.

**Answer:** Consider an e-commerce application:

```text
Normal traffic:
50 requests/second

Business-hour traffic:
200 requests/second

Known sale:
Every Friday at 20:00
```

A reasonable design could be:

| Requirement | Suitable policy |
|---|---|
| Keep CPU around 50% during normal operation | Target tracking |
| Add more capacity when CPU becomes severely overloaded | Step scaling |
| Prepare for a known recurring Friday event | Scheduled scaling |
| Prepare for recurring demand based on historical patterns | Predictive scaling |
| Basic legacy alarm → fixed capacity adjustment | Simple scaling |

In practice, you would not automatically configure every policy. Select the policy or combination that matches the workload's **traffic pattern, scaling signal, and response requirements**.

---
