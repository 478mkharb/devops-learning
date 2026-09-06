# Auto Scaling Group & Launch Template

## Interview Questions & Answers

### Q1. What is a Launch Template?

**Answer:** A Launch Template is a reusable EC2 launch specification. It can define the AMI, instance type, IAM instance profile, network interfaces/security groups, EBS mappings, user data, metadata options, and other launch settings. Versioning lets an ASG roll out configuration changes in a controlled way.

---

### Q2. What information belongs in a Launch Template?

**Answer:** A Launch Template is a reusable EC2 launch specification. It can define the AMI, instance type, IAM instance profile, network interfaces/security groups, EBS mappings, user data, metadata options, and other launch settings. Versioning lets an ASG roll out configuration changes in a controlled way.

---

### Q3. Why is a Launch Template preferred over a Launch Configuration?

**Answer:** A Launch Template is a reusable EC2 launch specification. It can define the AMI, instance type, IAM instance profile, network interfaces/security groups, EBS mappings, user data, metadata options, and other launch settings. Versioning lets an ASG roll out configuration changes in a controlled way.

---

### Q4. What is Launch Template versioning?

**Answer:** A Launch Template is a reusable EC2 launch specification. It can define the AMI, instance type, IAM instance profile, network interfaces/security groups, EBS mappings, user data, metadata options, and other launch settings. Versioning lets an ASG roll out configuration changes in a controlled way.

---

### Q5. How does an ASG select a Launch Template version?

**Answer:** A Launch Template is a reusable EC2 launch specification. It can define the AMI, instance type, IAM instance profile, network interfaces/security groups, EBS mappings, user data, metadata options, and other launch settings. Versioning lets an ASG roll out configuration changes in a controlled way.

---

### Q6. What is an Auto Scaling Group?

**Answer:** An Auto Scaling Group maintains a fleet of EC2 instances within minimum, desired, and maximum capacity limits. It can launch, replace, and terminate instances based on health and scaling decisions.

---

### Q7. Explain minimum, desired, and maximum capacity.

**Answer:** Minimum capacity is the lowest number of instances the ASG should maintain. Desired capacity is the current target number. Maximum capacity is the upper limit to which the ASG can scale.

---

### Q8. What is health-check replacement?

**Answer:** An ASG can use EC2 health checks and, when configured, ELB health checks. When an instance is considered unhealthy, the ASG terminates it and launches a replacement to restore desired capacity.

---

### Q9. How does an ASG replace an unhealthy instance?

**Answer:** Explain the Auto Scaling Group or launch mechanism in terms of capacity, instance lifecycle, health, scaling decisions, and how it interacts with Launch Templates, CloudWatch, and load balancers.

---

### Q10. What is a warm pool?

**Answer:** An ASG warm pool keeps pre-initialized EC2 instances outside the active InService capacity so they can enter service faster during scale-out. It is useful when application bootstrap is slow.

---

### Q11. What is target tracking scaling?

**Answer:** Target tracking scaling automatically adjusts ASG capacity to keep a selected metric near a target value, such as average CPU utilization or request count per target. It is generally the simplest choice when you want the fleet to maintain a stable metric target.

---

### Q12. What is step scaling?

**Answer:** Step scaling uses CloudWatch alarm breaches and different capacity adjustments for different breach ranges. For example, a large CPU breach can add more instances than a small breach.

---

### Q13. What is simple scaling?

**Answer:** Simple scaling applies one scaling adjustment after a CloudWatch alarm breach and historically relies on a cooldown period before another simple scaling action. Target tracking and step scaling are generally preferred for modern designs.

---

### Q14. What is scheduled scaling?

**Answer:** Scheduled scaling changes ASG capacity at known times. It is appropriate when demand is predictable, such as a business application that consistently receives heavy traffic during office hours.

---

### Q15. What is predictive scaling?

**Answer:** Predictive scaling uses historical usage patterns and forecasting to prepare capacity ahead of expected demand. It complements reactive scaling when traffic has recurring patterns.

---

### Q16. Compare target tracking, step, and scheduled scaling.

**Answer:** Target tracking scaling automatically adjusts ASG capacity to keep a selected metric near a target value, such as average CPU utilization or request count per target. It is generally the simplest choice when you want the fleet to maintain a stable metric target.

---

### Q17. Which policy is generally simplest for maintaining a target metric?

**Answer:** A CloudWatch metric is a time-ordered set of numerical measurements identified by a namespace and dimensions. Metrics can come from AWS services or be published as custom metrics.

---

### Q18. Can an ASG use multiple scaling policies?

**Answer:** Explain the Auto Scaling Group or launch mechanism in terms of capacity, instance lifecycle, health, scaling decisions, and how it interacts with Launch Templates, CloudWatch, and load balancers.

---

### Q19. What is a mixed instances policy?

**Answer:** A mixed instances policy lets an ASG use multiple EC2 instance types and purchasing options, commonly combining On-Demand and Spot capacity. It improves capacity flexibility and can reduce cost.

---

### Q20. How do On-Demand and Spot instances work together in an ASG?

**Answer:** An ASG can combine On-Demand and Spot capacity through a mixed instances policy. On-Demand provides more predictable capacity while Spot reduces cost but can be interrupted.

---

### Q21. What are termination policies?

**Answer:** ASG termination policies control which instances are selected during scale-in. Policies can consider Availability Zone balance, launch-template age, instance age, and other characteristics.

---

### Q22. What is instance refresh?

**Answer:** Instance Refresh replaces existing ASG instances with instances launched from an updated Launch Template configuration. It supports controlled rollouts using health thresholds, checkpoints, and warm-up settings.

---

### Q23. What are lifecycle hooks?

**Answer:** An ASG lifecycle hook pauses an instance during launch or termination so custom work can complete, such as bootstrapping, deregistration, log draining, or cleanup. The hook then completes or times out.

---

### Q24. How can lifecycle hooks delay termination or launch completion?

**Answer:** An ASG lifecycle hook pauses an instance during launch or termination so custom work can complete, such as bootstrapping, deregistration, log draining, or cleanup. The hook then completes or times out.

---

### Q25. What is a cooldown period?

**Answer:** Explain the Auto Scaling Group or launch mechanism in terms of capacity, instance lifecycle, health, scaling decisions, and how it interacts with Launch Templates, CloudWatch, and load balancers.

---

### Q26. What is default instance warmup?

**Answer:** Explain the Auto Scaling Group or launch mechanism in terms of capacity, instance lifecycle, health, scaling decisions, and how it interacts with Launch Templates, CloudWatch, and load balancers.

---
