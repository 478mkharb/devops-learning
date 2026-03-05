# Difference: VPC Peering vs Interface Endpoint vs Gateway Endpoint (AWS Networking)

This comparison is commonly asked in **DevOps / Cloud Engineer interviews** because all three enable **private connectivity inside AWS**, but they solve different problems.

---

# Quick Comparison Table

| Feature              | VPC Peering                | Interface Endpoint (PrivateLink)     | Gateway Endpoint                     |
| -------------------- | -------------------------- | ------------------------------------ | ------------------------------------ |
| Purpose              | Connect two VPCs privately | Connect VPC to AWS service privately | Connect VPC to S3/DynamoDB privately |
| Communication        | VPC ↔ VPC                  | VPC → AWS Service                    | VPC → AWS Service                    |
| Uses Private Network | Yes                        | Yes                                  | Yes                                  |
| Uses ENI             | No                         | Yes (Elastic Network Interface)      | No                                   |
| Route Table Required | Yes                        | No                                   | Yes                                  |
| Security Groups      | Uses instance SG           | Uses SG on endpoint ENI              | Uses endpoint policy                 |
| Supported Services   | Any VPC resources          | Most AWS services                    | Only S3 and DynamoDB                 |
| Transitive Routing   | Not supported              | N/A                                  | N/A                                  |

---

# 1. VPC Peering

## Definition

VPC Peering creates a **direct private network connection between two VPCs** so resources can communicate using **private IP addresses**.

## Flow Diagram

```
VPC-A (10.0.0.0/16)
      │
      │  VPC Peering Connection
      │
VPC-B (10.1.0.0/16)
```

## Traffic Flow

```
EC2 (VPC-A)
     │
     ▼
VPC Peering
     │
     ▼
EC2 (VPC-B)
```

## Key Points

* Uses **private IP communication**
* Requires **route table updates**
* **Non-transitive routing**

Example limitation:

```
VPC-A ↔ VPC-B
VPC-B ↔ VPC-C

VPC-A cannot reach VPC-C
```

---

# 2. Interface Endpoint (AWS PrivateLink)

## Definition

An **Interface Endpoint** allows instances in a VPC to connect to AWS services using **private IP addresses without using the internet**.

It creates an **Elastic Network Interface (ENI)** inside the subnet.

## Architecture

```
EC2 Instance
     │
     │ Private IP
     ▼
Interface Endpoint (ENI)
     │
     ▼
AWS Service
```

## Example Services

* Systems Manager (SSM)
* CloudWatch
* SNS
* SQS
* API Gateway
* Secrets Manager

## Key Points

* Uses **AWS PrivateLink**
* Creates **ENI in subnet**
* Uses **security groups**

---

# 3. Gateway Endpoint

## Definition

A **Gateway Endpoint** allows private access from a VPC to **Amazon S3 or DynamoDB** without using the internet.

Unlike Interface Endpoints, it is **implemented through route tables**.

## Architecture

```
EC2 Instance
     │
     ▼
Route Table
     │
     ▼
Gateway Endpoint
     │
     ▼
Amazon S3 / DynamoDB
```

## Route Table Example

```
pl-xxxx → Gateway Endpoint
```

## Key Points

* Works only for **S3 and DynamoDB**
* Configured through **route tables**
* No ENI is created

---

# DevOps Interview Scenario

## Without Endpoint

```
EC2 → NAT Gateway → Internet → S3
```

Problems:

* NAT cost
* Internet exposure

## With Gateway Endpoint

```
EC2 → Gateway Endpoint → S3
```

Benefits:

* Private network
* No NAT cost
* Better security

---

# Memory Trick

```
VPC Peering
VPC ↔ VPC communication

Interface Endpoint
VPC → AWS Service via ENI

Gateway Endpoint
VPC → S3 / DynamoDB via Route Table
```
