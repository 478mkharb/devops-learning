# AWS EC2: Achieving Persistent (Static) Private IP Using ENI

## 🎯 Objective

Ensure an EC2 instance can retain the **same private IP address even after termination** by leveraging **Elastic Network Interfaces (ENI)**.

---

## ⚠️ Important Concept

* Private IP is tied to **ENI (Elastic Network Interface)**, NOT the EC2 instance.
* Default EC2 launch creates an **auto-managed ENI** → deleted on termination.
* Solution: **Create and manage your own ENI manually**.

---

# 🧱 Architecture Overview

EC2 Instance  ⇄  ENI (with fixed private IP)

Terminate EC2 → ENI remains → Reattach ENI → Same IP restored

---

# 🚀 Step-by-Step Procedure (AWS Console)

## 🔹 Step 1: Create a Custom ENI

1. Go to AWS Console
2. Navigate to:
   EC2 → Network Interfaces
3. Click: **Create network interface**

### Fill Details:

* **Subnet**: Choose desired subnet (IMPORTANT: cannot change later)
* **Availability Zone**: Auto from subnet
* **Private IP**:

  * Select: "Assign a specific IP"
  * Enter your desired private IP (e.g., 10.0.1.50)
* **Security Groups**: Attach required SG

4. Click: **Create network interface**

✅ Result: ENI created with fixed private IP

---

## 🔹 Step 2: Launch EC2 Using Existing ENI

1. Go to:
   EC2 → Instances → Launch Instance

2. Configure normally (AMI, instance type, key pair)

3. In **Network Settings**:

   * Expand Advanced Network Configuration
   * Under Network Interfaces:

     * Select: **Existing network interface**
     * Choose the ENI you created

4. Launch instance

✅ Result: EC2 uses your ENI → gets same private IP

---

## 🔹 Step 3: Verify Private IP

1. Go to EC2 → Instances
2. Select your instance
3. Check:

   * Private IPv4 address

It should match the IP assigned to ENI

---

## 🔁 Step 4: Terminate and Reuse

### Terminate Instance

1. Select instance
2. Click: Instance State → Terminate

⚠️ Important:

* ENI will **NOT be deleted** because it was manually created

---

### Reattach ENI to New Instance

1. Launch new EC2 instance

2. In Network Settings:

   * Select: Existing network interface
   * Choose same ENI

3. Launch

✅ Result: New instance gets SAME private IP

---

# 🔒 Constraints & Rules

| Constraint           | Explanation                                    |
| -------------------- | ---------------------------------------------- |
| Same Subnet          | ENI cannot move across subnets                 |
| Same AZ              | Must launch instance in same Availability Zone |
| Single Attachment    | One ENI → one instance at a time               |
| Instance Type Limits | Some instance types limit number of ENIs       |

---

# ⚡ Alternative Approach (Secondary IP)

Instead of replacing ENI:

1. Assign **secondary private IP** to ENI
2. Use that IP in your application

Pros:

* Easier migration
* Can reassign between ENIs

---

# 🧠 DevOps Best Practices

Avoid relying on IP whenever possible.

Use:

* Internal DNS (Route 53 private hosted zones)
* Load Balancers (ALB/NLB)
* Service discovery (Consul, Kubernetes)

Reason:

* IP-based coupling = fragile systems

---

# 🛠️ Troubleshooting

### ❌ ENI not selectable during launch

* Ensure instance is in same AZ

### ❌ IP already in use

* Verify no other ENI is using that IP

### ❌ Cannot attach ENI

* Instance type ENI limit exceeded

---

# 🧩 Summary

| Goal                      | Solution              |
| ------------------------- | --------------------- |
| Persistent private IP     | Use custom ENI        |
| Reuse after termination   | Reattach ENI          |
| Production-grade approach | Use DNS instead of IP |

---
