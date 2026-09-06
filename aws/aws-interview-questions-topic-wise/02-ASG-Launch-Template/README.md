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

---

### Q12. What is step scaling?

**Answer:** Step scaling uses CloudWatch alarm breaches and different scaling adjustments for different ranges of metric deviation. For example, a severe CPU breach can add more instances than a small breach.

---

### Q13. What is simple scaling?

**Answer:** Simple scaling performs a single scaling adjustment when a CloudWatch alarm is breached and historically relies on a cooldown period before another simple scaling action. Target tracking and step scaling are generally preferred for modern ASGs.

---

### Q14. What is scheduled scaling?

**Answer:** Scheduled scaling changes ASG capacity at predefined times. It is appropriate when demand is predictable, such as a workload that consistently needs more instances during business hours.

---

### Q15. What is predictive scaling?

**Answer:** Predictive scaling uses historical utilization patterns and forecasting to prepare EC2 capacity ahead of expected demand. It is useful for recurring traffic patterns and complements reactive scaling policies.

---

### Q16. Compare target tracking, step, and scheduled scaling.

**Answer:** Target tracking automatically adjusts capacity toward a metric target. Step scaling applies different capacity adjustments based on the size of an alarm breach. Scheduled scaling changes capacity at predetermined times. Target tracking is usually best for a stable metric target; step scaling is useful for explicit thresholds; scheduled scaling is best for predictable demand.

---

### Q17. Which policy is generally simplest for maintaining a target metric?

**Answer:** A CloudWatch metric is a time series of numerical measurements identified by a namespace and dimensions. AWS services publish metrics and applications can publish custom metrics.

---

### Q18. Can an ASG use multiple scaling policies?

**Answer:** Yes. An ASG can have multiple scaling policies, for example target tracking for CPU utilization and another policy for request count. The policies operate within the group's minimum and maximum capacity limits, and scale-in behavior is coordinated to avoid unnecessarily aggressive reduction.

---

### Q19. What is a mixed instances policy?

**Answer:** A mixed instances policy allows an ASG to launch multiple EC2 instance types and combine purchasing options such as On-Demand and Spot. This improves capacity flexibility and can reduce cost.

---

### Q20. How do On-Demand and Spot instances work together in an ASG?

**Answer:** A mixed instances policy lets an ASG combine On-Demand and Spot instances. You can specify a base amount of On-Demand capacity and use Spot for additional capacity, with allocation strategies selecting suitable Spot pools. This balances availability and cost.

---

### Q21. What are termination policies?

**Answer:** ASG termination policies determine which instances are selected when the group needs to scale in. They can consider factors such as Availability Zone balance, launch-template version or configuration age, and instance age. The goal is to remove capacity while maintaining a balanced and predictable fleet.

---

### Q22. What is instance refresh?

**Answer:** Instance Refresh replaces existing ASG instances with instances launched from a new Launch Template configuration. It supports controlled rollouts using health thresholds, warm-up, checkpoints, and rollback-related controls.

---

### Q23. What are lifecycle hooks?

**Answer:** A lifecycle hook pauses an EC2 instance during an ASG launch or termination transition. This gives time for custom actions such as bootstrapping, registration, connection draining, log collection, or cleanup before the instance continues to InService or terminates.

---

### Q24. How can lifecycle hooks delay termination or launch completion?

**Answer:** During launch, a lifecycle hook can keep an instance in a pending lifecycle state while initialization or registration completes. During termination, it can keep the instance in a terminating state while connections are drained or cleanup runs. The lifecycle action must eventually be completed or it will time out according to the hook configuration.

---

### Q25. What is a cooldown period?

**Answer:** A scaling cooldown is a period intended to allow a previous scaling action to take effect before another simple scaling action occurs. Modern target tracking uses instance warmup to help prevent new instances from skewing scaling decisions.

---

### Q26. What is default instance warmup?

**Answer:** Default instance warmup is the ASG setting that specifies how long a newly launched instance is considered to be warming up before its metrics are fully considered for scaling decisions.

---
