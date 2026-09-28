# AWS Scenario-Based Interview Questions & Answers


# Level 1 — Intermediate

## 1. EC2 — Public EC2 Cannot Be Reached

### Question
You launch an EC2 instance in a public subnet, but you cannot SSH into it from your laptop. Walk through everything you would check.

### Answer
I would troubleshoot from the network outward:

1. Confirm the instance is running.
2. Confirm it has a public IPv4 address or Elastic IP.
3. Check that the subnet's route table has `0.0.0.0/0 → Internet Gateway`.
4. Confirm the VPC has an Internet Gateway attached.
5. Check the EC2 security group allows TCP 22 from my public IP.
6. Check the subnet NACL for inbound SSH and outbound ephemeral traffic.
7. Check the instance OS firewall and SSH service.
8. Confirm the correct username/key pair.
9. Check whether the instance is actually listening on port 22.

---

## 2. EC2 — Public IP but No Internet

### Question
An EC2 instance has a public IPv4 address but cannot access the Internet. What would you check?

### Answer
I would verify the subnet is public by checking its route table for:

```text
0.0.0.0/0 → Internet Gateway
```

Then I would verify the Internet Gateway is attached to the VPC, the security group permits outbound traffic, the NACL permits the traffic and return traffic, and the OS has working DNS and networking.

---

## 3. EC2 — Private Server Needs Internet

### Question
An EC2 instance is in a private subnet. It needs to download packages from the Internet but must not be directly reachable from the Internet. How would you design this?

### Answer
I would place the EC2 instance in a private subnet whose route table contains:

```text
0.0.0.0/0 → NAT Gateway
```

The NAT Gateway would be placed in a public subnet with:

```text
0.0.0.0/0 → Internet Gateway
```

The private instance can initiate outbound connections through NAT, but unsolicited Internet traffic cannot directly initiate a connection to the private EC2.

---

## 4. EC2 — Application Works Locally but Not Externally

### Question
An application listens on port 8080. `curl localhost:8080` works on EC2, but users cannot access it. What do you investigate?

### Answer
I would check:

- Application bind address
- EC2 security group
- NACL
- Route table
- Internet Gateway or ALB path
- Correct port
- OS firewall

If the application listens only on `127.0.0.1`, external traffic will not reach it. It generally needs to listen on the appropriate interface, such as `0.0.0.0`, while network access remains restricted through security controls.

---

## 5. EC2 — One Instance Cannot Reach Another

### Question
Two EC2 instances are in the same VPC, but one cannot communicate with the other. What would you check?

### Answer
I would check:

1. Destination private IP/DNS.
2. Security groups on both instances.
3. Subnet route tables.
4. NACLs.
5. OS firewalls.
6. Whether the application is listening on the destination port.

The VPC local route normally provides routing between subnets in the same VPC, but security and host-level controls can still block traffic.

---

## 6. EC2 — Public IP Changed

### Question
An EC2 instance was stopped and started, and its old public IP no longer works. Why?

### Answer
A normal auto-assigned public IPv4 address can change after a stop/start. For a stable public IP, I would use an **Elastic IP** where appropriate, although for production applications I would generally prefer exposing an ALB rather than directly exposing EC2.

---

## 7. EC2 — No Public IPs in Production

### Question
Production EC2 instances must never have public IP addresses, but administrators need access. How would you design this?

### Answer
I would keep the instances in private subnets. For administration, I would prefer **AWS Systems Manager Session Manager** with the required IAM role and network connectivity to Systems Manager endpoints. A controlled bastion architecture is another option when appropriate.

---

## 8. EC2 — Access 50 Private Servers

### Question
You have 50 private EC2 instances and administrators need shell access without public IPs. What AWS solution would you use?

### Answer
I would prefer **Systems Manager Session Manager**. It avoids exposing SSH to the Internet and provides centralized access through IAM. The instances need the SSM agent, an appropriate IAM role, and connectivity to the Systems Manager service, either through NAT or appropriate VPC endpoints.

---

# AMI

## 9. AMI — Clone Application Servers

### Question
You configured an EC2 server with the application, OS packages, agents, and configuration. You need 20 identical servers. What would you use?

### Answer
I would create a custom **AMI** from the configured EC2 instance and use it as the image in a Launch Template or when launching additional instances. This provides a repeatable server baseline.

---

## 10. AMI — Application Fails After Launch

### Question
You create an AMI and launch another EC2 instance from it, but the application does not start. What would you investigate?

### Answer
I would check application startup configuration, systemd services, environment variables, file permissions, instance-specific configuration, network dependencies, IAM role differences, and application logs. An AMI captures the image state but does not automatically guarantee that all runtime dependencies are correct.

---

## 11. AMI — Standard Production Image

### Question
Your company wants every production EC2 instance to use a standardized operating system and security configuration. How can AMIs help?

### Answer
I would create a hardened golden AMI containing the approved OS packages, agents, configuration, and security baseline. New instances would launch from that AMI, giving the organization a consistent starting point.

---

## 12. AMI — Move Workload

### Question
You need to recreate an EC2 workload in another Availability Zone using the same configuration. How can an AMI help?

### Answer
I can create an AMI from the existing instance and launch a new instance from that AMI in the required Availability Zone, while attaching the appropriate security groups, subnet, IAM role, and storage configuration.

---

# EBS

## 13. EBS — Disk Full

### Question
The root EBS volume is almost full. How would you increase capacity?

### Answer
I would modify the EBS volume to increase its size. After the AWS-side expansion, I would extend the partition and filesystem inside the operating system if required.

---

## 14. EBS — Data Must Survive EC2 Termination

### Question
An EC2 instance may be terminated, but important application data must survive. What would you do?

### Answer
I would store important persistent data on a separate EBS volume configured not to be deleted with the instance, or use an appropriate managed storage service. I would also use EBS snapshots for backup and recovery.

---

## 15. EBS — High I/O Workload

### Question
An application performs heavy disk I/O. How would you choose an EBS volume?

### Answer
I would examine required IOPS, throughput, latency, capacity, and workload characteristics. I would select an appropriate EBS type, such as a provisioned-IOPS volume when predictable high IOPS is required.

---

## 16. EBS — Move Data to Another EC2

### Question
You need to make data from an EBS volume available to another EC2 instance. What would you do?

### Answer
Depending on the requirement, I could detach and attach the EBS volume to another compatible instance in the same Availability Zone, create a snapshot and create a new volume from it, or use shared storage such as EFS if multiple instances need simultaneous access.

---

# EFS

## 17. EFS — Shared Storage

### Question
Ten EC2 instances in different Availability Zones need to read and write the same files. EBS or EFS?

### Answer
I would use **EFS** because it provides a shared network file system that can be mounted by multiple EC2 instances across Availability Zones.

---

## 18. EFS — Mount Fails

### Question
An EC2 instance cannot mount EFS. What would you check?

### Answer
I would check:

- EFS mount target exists in the relevant AZ.
- EC2 can resolve the EFS DNS name.
- EC2 security group permits outbound NFS.
- EFS mount-target security group allows inbound TCP 2049 from the EC2 security group.
- Route tables and NACLs permit connectivity.
- The EFS filesystem is available.

---

## 19. EFS — Missing Mount Targets

### Question
EFS has a mount target in only one AZ while application servers run in three AZs. What concern does this create?

### Answer
The architecture may introduce unnecessary cross-AZ traffic and reduce resilience. I would normally create EFS mount targets in the AZs where the application needs local access.

---

## 20. EFS — Auto Scaling Application

### Question
Application servers are dynamically created and terminated, but all instances need access to the same uploaded files. How would you design storage?

### Answer
I would use EFS for shared filesystem data or S3 for object data depending on the application. This prevents application instances from depending on local instance storage.

---

# Auto Scaling

## 21. Auto Scaling — Traffic Spike

### Question
An EC2 application behind an ALB receives a sudden traffic increase. How would Auto Scaling respond?

### Answer
I would configure an Auto Scaling Group with a Launch Template and scaling policies. The ASG could scale based on CPU, ALB request count per target, or another appropriate metric.

---

## 22. Auto Scaling — Instances Constantly Replaced

### Question
An ASG keeps launching instances and then terminating them. What would you investigate?

### Answer
I would check:

- EC2 health
- ALB health checks
- ASG health-check configuration
- Startup failures
- User data
- IAM permissions
- Application logs
- Instance status checks
- Launch Template configuration

---

## 23. Auto Scaling — ALB Unhealthy

### Question
The ASG says instances are healthy, but the ALB says targets are unhealthy. What could cause this?

### Answer
I would check the ALB health-check path, port, protocol, target security group, application bind address, application health endpoint, and whether the ALB can route to the target subnet.

---

## 24. Auto Scaling — Multi-AZ

### Question
You want application instances distributed across three Availability Zones. How would you configure the ASG?

### Answer
I would configure the ASG with subnets from all three AZs and appropriate desired/minimum/maximum capacity. The ALB would also use subnets across multiple AZs.

---

## 25. Auto Scaling — Request-Based Scaling

### Question
CPU stays low but users experience high latency during traffic spikes. How could you scale based on application demand?

### Answer
I would consider an ALB metric such as **RequestCountPerTarget** or a suitable custom CloudWatch metric. Scaling based on the workload's actual bottleneck is better than assuming CPU is always the correct signal.

---

# ALB / NLB

## 26. ALB — Microservice Routing

### Question
Three services use `/employee`, `/salary`, and `/attendance`. They run on private EC2 instances. How would you expose them through one ALB?

### Answer
I would use an **Application Load Balancer** with path-based listener rules:

```text
/employee/*   → Employee target group
/salary/*     → Salary target group
/attendance/* → Attendance target group
```

The EC2 instances can remain private.

---

## 27. ALB — Public ALB, Private EC2

### Question
How can a public ALB send traffic to EC2 instances that have no public IP?

### Answer
The ALB has network connectivity to the private target subnets. Clients connect to the ALB, and the ALB establishes a separate connection to the private targets. The target security group should allow the application port from the ALB security group.

---

## 28. NLB — TCP Workload

### Question
Your application requires high-performance TCP traffic and does not need HTTP path-based routing. Which load balancer would you consider?

### Answer
I would consider a **Network Load Balancer**, which operates at Layer 4 and is designed for TCP/UDP/TLS workloads and high connection performance.

---

## 29. ALB — 502 Error

### Question
The ALB is reachable, but clients receive HTTP 502. How would you troubleshoot?

### Answer
I would check target health, application listener port, security groups, application response behavior, target connection errors, health-check configuration, and application logs.

---

## 30. NLB — Connection Accepted but Application Fails

### Question
An NLB accepts connections but the backend application does not respond. What would you check?

### Answer
I would check target health, listener/target ports, target security groups, NACLs, route tables, application listening state, and application logs.

---

# VPC / Subnets / Routing

## 31. VPC — Three-Tier Design

### Question
Design a production VPC with public ALB, private application servers, and private database servers.

### Answer
I would use multiple AZs and create:

```text
Public subnets
  → ALB

Private application subnets
  → EC2 / ECS / EKS

Private database subnets
  → RDS / Aurora
```

Public subnets route Internet-bound traffic to the Internet Gateway. Private application subnets can route outbound Internet traffic through NAT Gateway. Database subnets generally have no direct Internet route.

---

## 32. VPC — What Makes a Subnet Public?

### Question
What specifically makes a subnet public?

### Answer
A subnet is considered public when its associated route table provides a route to an Internet Gateway, typically:

```text
0.0.0.0/0 → Internet Gateway
```

An instance also needs an appropriate public IPv4 address/EIP and security controls for direct Internet communication.

---

## 33. Route Table — Same VPC Communication

### Question
Two EC2 instances are in different subnets of the same VPC. How can they communicate?

### Answer
The VPC route table normally contains the VPC CIDR with the `local` target, allowing routing between subnets in the VPC. Security groups and NACLs still need to permit the traffic.

---

## 34. Route Table — Private EC2 Has No Internet

### Question
A private EC2 instance has outbound HTTPS allowed but cannot reach the Internet. What do you check?

### Answer
I would verify:

```text
Private subnet route table
0.0.0.0/0 → NAT Gateway

NAT Gateway
→ Public subnet

Public subnet route table
0.0.0.0/0 → Internet Gateway
```

Then I would check NACLs, security groups, DNS, and the NAT Gateway state.

---

## 35. Route Table — Wrong Association

### Question
You accidentally associate the wrong route table with a private subnet. What could happen?

### Answer
The subnet may lose required routes, accidentally gain Internet access through an IGW, send traffic through the wrong NAT Gateway, or lose connectivity to other networks.

---

# Internet Gateway

## 36. Internet Gateway — Private EC2

### Question
Can a private EC2 instance directly use an Internet Gateway for outbound Internet access?

### Answer
A private subnet should not use the Internet Gateway as its direct default route. For outbound Internet access, I would normally use a NAT Gateway in a public subnet. The private subnet's default route points to the NAT Gateway.

---

## 37. Internet Gateway — Public EC2

### Question
An EC2 instance has a public IP but cannot access the Internet. What role does the IGW play?

### Answer
The VPC needs an attached Internet Gateway and the subnet route table needs a route such as:

```text
0.0.0.0/0 → Internet Gateway
```

The instance also needs a public address and appropriate security/NACL rules.

---

# NAT Gateway

## 38. NAT Gateway — Complete Flow

### Question
Explain the traffic flow when private EC2 accesses `https://example.com`.

### Answer
The flow is approximately:

```text
Private EC2
  ↓
Private subnet route table
  ↓
NAT Gateway
  ↓
Public subnet route table
  ↓
Internet Gateway
  ↓
Internet
```

The return traffic comes back through the NAT Gateway to the private instance.

---

## 39. NAT Gateway — Three AZs

### Question
You have private application subnets in three AZs but only one NAT Gateway. What are the concerns?

### Answer
The NAT Gateway can be a cross-AZ dependency for the other AZs. A failure or disruption affecting its AZ can impact outbound connectivity for other AZs, and cross-AZ traffic can add cost. For stronger AZ isolation, I would generally consider one NAT Gateway per AZ.

---

## 40. NAT Gateway — Doesn't Receive Inbound Internet Connections

### Question
Why can't an Internet user initiate a connection to a private EC2 through a NAT Gateway?

### Answer
A NAT Gateway is designed for outbound connections initiated from private resources. It is not a public inbound load balancer or reverse proxy for arbitrary Internet-initiated connections.

---

# Security Groups / NACLs

## 41. ALB-to-EC2 Security Groups

### Question
How would you configure security groups when only the ALB should access the application EC2 instances?

### Answer
I would configure:

```text
ALB SG:
Inbound 443 from Internet
Outbound application traffic

EC2 SG:
Inbound 8080 from ALB SG
Outbound as required
```

Using the ALB security group as the source avoids hardcoding ALB IP ranges.

---

## 42. Security Group vs NACL

### Question
Explain a practical difference between a Security Group and a NACL.

### Answer
A security group is stateful and is associated with ENIs/resources. A NACL is associated with subnets and is stateless, so return traffic must be explicitly allowed by the NACL rules.

---

## 43. NACL — Return Traffic Fails

### Question
Inbound HTTPS is allowed in a NACL, but connections still fail. What could be missing?

### Answer
Because NACLs are stateless, the return traffic must also be permitted. I would check outbound ephemeral ports and the corresponding inbound/outbound rules required by the connection.

---

## 44. Security Group Works with 0.0.0.0/0

### Question
The application works when the security group allows `0.0.0.0/0`, but fails when restricted to a specific security group. What would you investigate?

### Answer
I would verify that the source security group is actually attached to the connecting resource and that the traffic path is what I expect. I would also verify the correct port, target ENI, load balancer path, and whether another component such as a NACL is blocking traffic.

---

# VPC Peering / Transit Gateway

## 45. VPC Peering — Basic Connectivity

### Question
VPC-A is `10.0.0.0/16` and VPC-B is `10.1.0.0/16`. An EC2 in A must communicate with an EC2 in B. What is required?

### Answer
I would:

1. Create the VPC peering connection.
2. Accept it from the other side.
3. Add routes in VPC-A for `10.1.0.0/16 → peering connection`.
4. Add routes in VPC-B for `10.0.0.0/16 → peering connection`.
5. Update security groups.
6. Check NACLs.

---

## 46. VPC Peering — Overlapping CIDRs

### Question
Two VPCs use the same CIDR. Can you use VPC Peering for normal IP communication between them?

### Answer
Overlapping CIDRs create ambiguous routing and prevent normal VPC peering connectivity between those address spaces. I would redesign the network addressing before establishing the desired private connectivity.

---

## 47. Transit Gateway — Many VPCs

### Question
Your organization has 40 VPCs. Full-mesh VPC peering has become difficult to manage. What would you consider?

### Answer
I would consider **AWS Transit Gateway** as a central network hub. VPCs attach to the Transit Gateway, and Transit Gateway route tables control which attached networks can communicate.

---

## 48. Transit Gateway — Segmentation

### Question
Production and Development VPCs should both access a Shared Services VPC, but Production and Development must not communicate directly. How could Transit Gateway help?

### Answer
I would use separate Transit Gateway route tables or routing domains and control propagation/associations so Production has a route to Shared Services but not Development, and Development has a route to Shared Services but not Production.

---

# Route 53 / CloudFront / S3

## 49. Route 53 — Private DNS

### Question
Private EC2 instances need to access an internal application using `employee.internal.example.com`. How would you implement this?

### Answer
I would create a **Route 53 private hosted zone**, associate it with the relevant VPC, and create the required internal DNS record.

---

## 50. CloudFront — Global Application

### Question
Your application is hosted in one AWS Region but has global users. How would CloudFront help?

### Answer
CloudFront can cache appropriate content at edge locations closer to users, reducing latency and reducing requests reaching the origin.

---

## 51. S3 — Static Website

### Question
You need to host a static React application using AWS. What architecture would you use?

### Answer
I would store the built static assets in **S3** and use **CloudFront** for HTTPS and edge delivery. I would configure the origin access mechanism so users access the content through the intended CloudFront path rather than exposing the bucket unnecessarily.

---

# RDS / Aurora

## 52. RDS — Private Database

### Question
How would you ensure RDS is not directly accessible from the Internet?

### Answer
I would place RDS in private subnets, disable public accessibility, and configure the database security group to allow the database port only from the application security group.

---

## 53. RDS — Application Cannot Connect

### Question
An EC2 application cannot connect to RDS on port 5432. RDS is healthy. What would you check?

### Answer
I would check:

- RDS endpoint and port
- RDS security group
- EC2 security group
- Subnet route tables
- NACLs
- DNS resolution
- Database listener/configuration
- Application credentials

---

## 54. Aurora — High Availability

### Question
A production application needs a highly available relational database with read scaling. Why might Aurora be considered?

### Answer
Aurora provides a managed relational database architecture with high availability and read-replica capabilities. I would evaluate Aurora against RDS engines based on compatibility, workload, performance, and cost requirements.

---

## 55. Aurora — Read-Heavy Application

### Question
An application has heavy database reads and relatively few writes. How could Aurora help?

### Answer
I could use Aurora replicas for read traffic and direct application reads to an appropriate reader endpoint, while writes go to the writer endpoint.

---

# Lambda / API Gateway

## 56. Lambda — Serverless API

### Question
You need a serverless REST API that stores data in DynamoDB. Which AWS services would you use?

### Answer
A common architecture would be:

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
DynamoDB
```

API Gateway handles API requests and Lambda executes the business logic.

---

## 57. Lambda — VPC Access

### Question
A Lambda function needs to access a private RDS database. How would you design this?

### Answer
I would configure the Lambda function for VPC access using appropriate private subnets and security groups. The RDS security group would allow the database port from the Lambda security group.

I would also be careful about outbound Internet requirements because placing Lambda in a VPC changes how it reaches other resources and the Internet.

---

## 58. Lambda — Private RDS and Internet API

### Question
A Lambda function must access private RDS and also call an external Internet API. What networking architecture would you consider?

### Answer
I would place the Lambda function in appropriate private subnets and provide outbound Internet access through a NAT Gateway if the external API must be reached over the public Internet. RDS remains private and permits traffic from the Lambda security group.

---

## 59. API Gateway — Throttling

### Question
A public API is receiving excessive traffic and overwhelming the backend Lambda functions. How could AWS help?

### Answer
I would use **API Gateway throttling**, quotas where appropriate, and potentially AWS WAF rate-based rules. I would also configure Lambda concurrency controls where appropriate.

---

## 60. Lambda — Cold Start

### Question
A latency-sensitive Lambda API occasionally experiences cold-start latency. How would you address it?

### Answer
I would reduce initialization overhead and package size, optimize the runtime, and consider **Provisioned Concurrency** when predictable startup latency is important.

---

# ECS

## 61. ECS — Containerized Application

### Question
Your team has Dockerized an application and wants AWS-managed container orchestration without managing Kubernetes. What would you consider?

### Answer
I would consider **Amazon ECS**, and for serverless container infrastructure I could use **AWS Fargate** so I do not manage EC2 worker nodes.

---

## 62. ECS — Private Tasks

### Question
ECS tasks run in private subnets and need to pull images from ECR. What connectivity does the architecture need?

### Answer
The tasks need network connectivity to ECR and other required AWS services. This can be provided through NAT Gateway or appropriate VPC endpoints, depending on the services and desired architecture. The task execution role also needs the required ECR permissions.

---

## 63. ECS — Task Cannot Start

### Question
An ECS task repeatedly fails to start. What would you investigate?

### Answer
I would inspect ECS task events and stopped-task reasons, then check:

- Image availability
- ECR permissions
- Task execution role
- Task CPU/memory
- Environment variables
- Secrets
- Networking
- Security groups
- Container logs
- Health checks

---

## 64. ECS — Service Behind ALB

### Question
You run an ECS service behind an ALB. How should security groups be configured?

### Answer
The ALB security group should allow the required client traffic, such as HTTPS. The ECS task security group should allow the application port only from the ALB security group.

---

## 65. ECS — Auto Scaling

### Question
An ECS service experiences increased traffic. How can it automatically add containers?

### Answer
I would configure ECS Service Auto Scaling using metrics such as CPU, memory, or ALB request count per target. The desired task count can increase as demand grows.

---

# ECR

## 66. ECR — Store Container Images

### Question
Your ECS application needs a private Docker image repository. Which AWS service would you use?

### Answer
I would use **Amazon ECR**. ECS can pull images from ECR using IAM-based authentication and permissions.

---

## 67. ECR — ECS Cannot Pull Image

### Question
An ECS task cannot pull its ECR image. What would you check?

### Answer
I would check:

1. Repository and image/tag.
2. Task execution IAM role.
3. ECR permissions.
4. Network connectivity from the task.
5. NAT Gateway or required VPC endpoints.
6. Region configuration.
7. ECR repository policy if cross-account access is involved.

---

## 68. ECR — Image Security

### Question
You want to detect vulnerabilities in container images before deployment. How could ECR help?

### Answer
I would enable appropriate **ECR image scanning** and incorporate scan results into the deployment process. I would also use immutable tagging or digest-based deployment practices where appropriate.

---

# EKS

## 69. EKS — Why Kubernetes

### Question
Your organization already has Kubernetes expertise and wants to run Kubernetes workloads on AWS. Why might EKS be selected?

### Answer
EKS provides a managed Kubernetes control plane and integrates with AWS networking, IAM, load balancing, storage, and observability services while retaining Kubernetes APIs and deployment patterns.

---

## 70. EKS — Pods in Private Subnets

### Question
Your EKS worker nodes run in private subnets. Pods need to communicate with AWS services. What networking considerations would you investigate?

### Answer
I would examine the EKS VPC configuration, pod networking, route tables, security groups, NAT Gateway or VPC endpoints, and IAM permissions. The exact requirements depend on how the cluster and pods are configured.

---

## 71. EKS — Pod Cannot Reach RDS

### Question
An EKS application cannot connect to private RDS. What would you investigate?

### Answer
I would verify:

- RDS security group
- Pod/node security group
- RDS port
- VPC routing
- NACLs
- DNS
- EKS networking configuration
- Application connection configuration

The RDS security group should allow the required database port from the appropriate EKS source.

---

## 72. EKS — Internet Access from Private Nodes

### Question
EKS nodes are in private subnets and pods need to download external dependencies. How would you provide outbound Internet access?

### Answer
I would normally route private subnet Internet-bound traffic through a NAT Gateway. For AWS service access such as ECR and S3, I would evaluate VPC endpoints to avoid unnecessary NAT traffic.

---

## 73. EKS — LoadBalancer Service

### Question
You deploy an application in EKS and need to expose it externally. Which AWS load balancing integration would you consider?

### Answer
I would use the AWS Load Balancer integration appropriate to the Kubernetes resource. For HTTP/HTTPS applications, an ALB-based ingress architecture is common; for Layer 4 services, an NLB can be appropriate.

---

## 74. EKS — Cluster Cannot Pull Image

### Question
An EKS pod is stuck because it cannot pull an image from ECR. What would you troubleshoot?

### Answer
I would check:

- ECR repository/image/tag
- Pod/node IAM permissions
- EKS workload identity configuration where applicable
- Node/pod network connectivity
- NAT Gateway or VPC endpoints
- Security groups
- DNS
- ECR repository policy

---

# IAM / KMS / CloudWatch / CloudTrail

## 75. IAM — EC2 Access to S3

### Question
An EC2 application needs to upload objects to one S3 bucket. How would you grant access?

### Answer
I would attach an IAM role to the EC2 instance with least-privilege permissions limited to the required S3 bucket and actions. I would avoid storing long-lived AWS access keys on the server.

---

## 76. IAM — ECS Task Access

### Question
An ECS application needs to read secrets and upload files to S3. How would you grant those permissions?

### Answer
I would use an ECS task role with only the required permissions. The task role should represent the application's AWS permissions and should not be broader than necessary.

---

## 77. KMS — Encryption

### Question
Your company requires encryption for sensitive data. How can KMS fit into the architecture?

### Answer
AWS KMS can manage encryption keys used by AWS services such as EBS, S3, RDS, and other services. I would combine encryption with IAM/key policies and appropriate access controls.

---

## 78. CloudWatch — EC2 Troubleshooting

### Question
An EC2 application suddenly becomes slow. What CloudWatch metrics would you inspect?

### Answer
I would examine CPU, network traffic, disk-related metrics available for the workload, status checks, and application logs. For memory and filesystem-level metrics, I would use the CloudWatch agent where required.

---

## 79. CloudTrail — Unknown Change

### Question
A production security group was modified and nobody knows who changed it. How would you investigate?

### Answer
I would use **CloudTrail** to find the API event for the security-group modification and identify the identity, timestamp, source, and API action associated with the change.

---

## 80. CloudWatch — ALB Troubleshooting

### Question
Users report intermittent HTTP errors from an ALB. What CloudWatch information would you correlate?

### Answer
I would examine ALB request count, target response time, HTTP 4xx/5xx metrics, target health, rejected connections, and corresponding application logs. I would correlate the timing with EC2, ECS, or EKS metrics.

---

# Advanced Networking & Private Access

## 81. Complete Three-Tier Network

### Question
Design a three-AZ production VPC with public ALB, private EC2, and private RDS.

### Answer
I would create:

```text
VPC: 10.0.0.0/16

AZ-a:
  Public subnet
  Private app subnet
  Private DB subnet

AZ-b:
  Public subnet
  Private app subnet
  Private DB subnet

AZ-c:
  Public subnet
  Private app subnet
  Private DB subnet
```

The public route tables use the Internet Gateway. Private application subnets use NAT for outbound Internet access. Database subnets remain private. The ALB spans public subnets, application servers span private subnets, and RDS/Aurora uses private database subnets.

---

## 82. Private EC2 — Administrator Access

### Question
A private EC2 instance has:

```text
Private IP = 10.0.20.15
Public IP = None
```

Its route table contains:

```text
10.0.0.0/16 → local
0.0.0.0/0 → NAT Gateway
```

An administrator wants to SSH to it directly from the Internet. Why will this not work, and what would you use instead?

### Answer
The instance has no public address and the private subnet has no direct Internet ingress path. NAT is for outbound connections initiated by private resources.

I would prefer **Systems Manager Session Manager**. A controlled bastion architecture or private connectivity such as VPN/Direct Connect can also be considered depending on the organization.

---

## 83. Private EC2 — SSM Access

### Question
You want administrators to access private EC2 instances using Systems Manager without public IPs. What AWS components are required?

### Answer
The instance needs:

- SSM Agent
- IAM instance role with required SSM permissions
- Network connectivity to Systems Manager endpoints
- Appropriate DNS/network configuration

Connectivity can be through NAT Gateway or VPC endpoints, depending on the architecture.

---

## 84. Private EC2 — No Internet Access at All

### Question
Security says private EC2 servers must not have general Internet access but must use SSM, CloudWatch, Secrets Manager, and S3. How would you design this?

### Answer
I would use **VPC endpoints** for the required AWS services where supported, such as S3 gateway endpoints and appropriate interface endpoints for services such as Systems Manager-related APIs, CloudWatch, and Secrets Manager. This avoids requiring general Internet access through NAT for those AWS services.

---

## 85. ALB to Private EC2 — Complete Troubleshooting

### Question
Users can reach the public ALB, but its private EC2 targets are unhealthy. Give a systematic troubleshooting process.

### Answer
I would check:

1. ALB listener.
2. Target group port/protocol.
3. Health-check path.
4. EC2 application listening address/port.
5. EC2 security group allowing traffic from ALB SG.
6. ALB subnet routing.
7. EC2 subnet routing/local VPC route.
8. NACL inbound/outbound rules.
9. OS firewall.
10. Application logs.

---

## 86. Private EC2 — Outbound Internet Failure

### Question
Private EC2 cannot access the Internet. The NAT Gateway is available. What do you check from the instance outward?

### Answer
I would check:

```text
EC2
 ↓
Security Group outbound
 ↓
Subnet route table
 ↓
NAT Gateway
 ↓
NAT public subnet route table
 ↓
Internet Gateway
 ↓
Internet
```

I would also verify NACLs, NAT subnet routing, Elastic IP association, DNS resolution, and NAT Gateway health.

---

## 87. VPC Peering — Full Troubleshooting

### Question
VPC peering is active, but EC2 in VPC-A cannot reach EC2 in VPC-B. What do you check?

### Answer
I would check:

1. Peering status.
2. Non-overlapping CIDRs.
3. Route from A to B through peering.
4. Route from B to A through peering.
5. Source/destination security groups.
6. NACLs.
7. OS firewall.
8. Correct private IP/DNS.
9. Application listener.

---

## 88. Transit Gateway — Selective Connectivity

### Question
You have Production, Development, and Shared Services VPCs. Production and Development may both access Shared Services, but neither may access the other. How would you design it?

### Answer
I would use Transit Gateway with separate routing domains/route tables. Production and Development would receive routes to Shared Services, while routes between Production and Development would be absent or explicitly prevented according to the TGW routing design.

---

## 89. Cross-Account VPC Connectivity

### Question
Production and Shared Services are in different AWS accounts. They need private network communication. What AWS architecture could you use?

### Answer
I could use Transit Gateway with cross-account attachments, or VPC Peering for a smaller/simple connectivity requirement. For many accounts and VPCs, Transit Gateway is generally easier to scale and centrally manage.

---

## 90. VPC Design — 40 VPCs

### Question
Your company has 40 VPCs and wants centralized private connectivity. What would you consider instead of creating a full mesh of VPC Peering connections?

### Answer
I would consider **Transit Gateway**. Each VPC attaches to the Transit Gateway, and centralized route tables can control connectivity between network segments.

---

# Hardest Scenarios

## 91. Complete AWS Production Architecture

### Question
Design an AWS architecture for:

```text
Internet
   ↓
Route 53
   ↓
CloudFront
   ↓
ALB
   ↓
Private ECS/EKS/EC2
   ↓
Private RDS/Aurora
```

The application must be highly available and private wherever possible.

### Answer
I would deploy the application across multiple AZs. Route 53 provides DNS, CloudFront provides edge delivery where appropriate, and the ALB distributes traffic to private application workloads. ECS/EKS/EC2 would run in private subnets, while RDS/Aurora would use private database subnets. NAT Gateways or VPC endpoints provide required outbound AWS/service connectivity.

---

## 92. ECS — Private Production Architecture

### Question
You need to run ECS tasks in private subnets. They must:

- Pull images from ECR.
- Access RDS.
- Upload files to S3.
- Call an external API.

How would you design networking?

### Answer
I would place ECS tasks in private subnets. The ECS task security group would allow database traffic to RDS and only required inbound traffic from the ALB. For ECR/S3/AWS service access, I would use appropriate VPC endpoints where practical. For the external public API, I would use NAT Gateway for outbound Internet access.

---

## 93. EKS — Private Cluster Workload

### Question
EKS worker nodes are private. Pods need ECR, S3, CloudWatch, and RDS access. They must not accept Internet traffic. How would you design it?

### Answer
I would run worker nodes in private subnets, expose applications through an AWS load balancer rather than public node IPs, allow RDS access through security groups, use VPC endpoints for AWS services where practical, and use NAT only for dependencies that genuinely require public Internet access.

---

## 94. Lambda — Private RDS + External API

### Question
A Lambda function must access private Aurora and an external payment API. What would you configure?

### Answer
I would configure Lambda VPC access in private subnets. Aurora would allow the Lambda security group on the database port. The private subnets would route Internet-bound traffic through NAT Gateway so Lambda can reach the external API.

I would also ensure the Lambda execution role contains only required permissions.

---

## 95. ECR — Cross-Account ECS Deployment

### Question
The ECR repository is in a central AWS account, while ECS runs in a production account. The ECS tasks cannot pull the image. What would you check?

### Answer
I would verify the ECR repository policy allows the production account/task execution role as required, the ECS task execution role has the necessary ECR permissions, the image URI/region is correct, and the ECS task has network connectivity to ECR.

---

## 96. Multi-AZ Failure

### Question
Your application is deployed across three AZs. One AZ completely fails. What AWS components must be designed correctly for the application to continue operating?

### Answer
I would ensure the ALB spans multiple AZs, the application workload is distributed across AZs through ASG/ECS/EKS, and the database has an appropriate Multi-AZ/high-availability architecture. I would also avoid making one AZ a mandatory dependency for NAT or other critical components unless intentionally designed.

---

## 97. NAT Gateway Failure

### Question
One NAT Gateway fails. Your private application servers in that AZ lose outbound Internet connectivity. How would you improve the architecture?

### Answer
I would consider deploying a NAT Gateway in each AZ and configuring each private subnet to use the NAT Gateway in its own AZ. This reduces cross-AZ dependency and improves AZ-level resilience.

---

## 98. Security Group + NACL Troubleshooting

### Question
A connection from ALB to EC2 is failing. The EC2 security group allows the ALB security group on port 8080. What could still block traffic?

### Answer
Possible blockers include:

- ALB security group
- EC2 security group
- NACL inbound rules
- NACL outbound rules
- Route tables
- Application listener
- OS firewall
- Incorrect target port
- Incorrect health-check path

The key point is that a correct security-group rule does not prove end-to-end network connectivity.

---

## 99. Full Private Server Access Design

### Question
You are told:

> "No production EC2 server may have a public IP. SSH must not be open to the Internet. Administrators must still have shell access. Servers must reach S3, CloudWatch, Secrets Manager, and Systems Manager."

Design the AWS solution.

### Answer
I would:

1. Place EC2 instances in private subnets.
2. Use IAM roles for the instances.
3. Use Systems Manager Session Manager for administrative access.
4. Provide VPC endpoints for required AWS services where appropriate.
5. Use S3 gateway endpoint for S3 where appropriate.
6. Use interface endpoints for services requiring them.
7. Keep security groups restricted.
8. Avoid exposing port 22 publicly.
9. Use CloudWatch for monitoring/logging.
10. Use KMS where encryption with customer-managed keys is required.

---

# 100. MASTER AWS NETWORKING SCENARIO

## Question

You are asked to design this production environment:

```text
                         Internet
                            |
                         Route 53
                            |
                        CloudFront
                            |
                           ALB
                            |
             +--------------+--------------+
             |              |              |
          AZ-A            AZ-B            AZ-C
             |              |              |
        Private EC2     Private EC2     Private EC2
             |              |              |
             +--------------+--------------+
                            |
                       RDS / Aurora
```

Requirements:

- EC2 must not have public IPs.
- RDS/Aurora must be private.
- ALB must be Internet-facing.
- EC2 needs outbound Internet access.
- Administrators need access to private EC2.
- The application must survive an AZ failure.
- EC2 needs S3 access.
- ECS/EKS workloads may also run in the private application subnets.
- Containers need ECR access.
- Lambda needs private database access.
- Production and Development are separate VPCs.
- Both VPCs need access to Shared Services.
- Production and Development must not communicate directly.
- Security groups and NACLs must enforce least privilege.
- The organization has multiple AWS accounts.

### Answer

I would design it as follows.

### 1. VPC

Use a dedicated production VPC, for example:

```text
10.0.0.0/16
```

Use non-overlapping CIDRs across all VPCs.

### 2. Three Availability Zones

Create:

```text
AZ-A:
  Public subnet
  Private application subnet
  Private database subnet

AZ-B:
  Public subnet
  Private application subnet
  Private database subnet

AZ-C:
  Public subnet
  Private application subnet
  Private database subnet
```

### 3. Internet Gateway

Attach one Internet Gateway to the VPC.

Public subnet route tables:

```text
10.0.0.0/16 → local
0.0.0.0/0   → Internet Gateway
```

### 4. ALB

Place the Internet-facing ALB in the public subnets across multiple AZs.

```text
Internet
   ↓
ALB :443
```

The ALB security group allows HTTPS from required Internet sources.

### 5. Private EC2 / ECS / EKS

Application workloads remain in private subnets.

```text
ALB
 ↓
Private EC2/ECS/EKS
```

The application security group allows application traffic only from the ALB security group.

### 6. NAT Gateway

For outbound Internet access, use NAT Gateway.

Ideally:

```text
AZ-A private subnet → NAT-A
AZ-B private subnet → NAT-B
AZ-C private subnet → NAT-C
```

Each NAT Gateway is in a corresponding public subnet.

### 7. RDS / Aurora

Database instances remain in private database subnets.

The database security group allows:

```text
DB port
Source = Application security group
```

It does not allow direct Internet access.

### 8. S3

For S3 access, use an S3 VPC endpoint where appropriate.

This allows private workloads to access S3 without unnecessarily traversing NAT.

### 9. ECR

For ECS/EKS image pulls, provide the required ECR connectivity using appropriate VPC endpoints and/or NAT depending on the exact AWS service/API requirements.

### 10. Lambda

If Lambda needs private Aurora/RDS access, configure Lambda VPC access and place it in appropriate private subnets.

Aurora security group:

```text
Inbound DB port
Source = Lambda security group
```

### 11. Private EC2 Administration

Use Systems Manager Session Manager rather than exposing SSH publicly.

```text
Administrator
      |
      ▼
Systems Manager
      |
      ▼
Private EC2
```

Use VPC endpoints or NAT for Systems Manager connectivity according to the network design.

### 12. Route 53

Use Route 53 for public DNS:

```text
app.example.com → CloudFront/ALB
```

For internal service discovery, use Route 53 private hosted zones where appropriate.

### 13. CloudFront

CloudFront can sit in front of the public application for edge delivery and caching where applicable:

```text
User
 ↓
Route 53
 ↓
CloudFront
 ↓
ALB
```

### 14. Production / Development / Shared Services

For multiple VPCs and AWS accounts, I would use Transit Gateway.

```text
                 Transit Gateway
                 /      |       \
                /       |        \
             PROD     DEV     SHARED
```

Use separate Transit Gateway route tables/segmentation so:

```text
PROD → SHARED       Allowed
DEV  → SHARED       Allowed
PROD → DEV          Not allowed
DEV  → PROD         Not allowed
```

### 15. Security Groups

Use security-group references wherever possible:

```text
Internet
   ↓
ALB SG
   ↓
Application SG
   ↓
Database SG
```

This creates a clear trust chain.

### 16. NACLs

Use subnet-level NACLs as an additional network control layer. Because NACLs are stateless, both directions of required traffic must be permitted.

### 17. AZ Failure

If AZ-A fails:

```text
AZ-B + AZ-C
```

continue serving application traffic through the ALB.

The ASG/ECS/EKS deployment should maintain capacity across the remaining AZs.

### 18. NAT Failure

Using one NAT Gateway per AZ avoids making one AZ's NAT Gateway a single outbound dependency for all application subnets.

### 19. Request Flow

A normal user request is:

```text
User
 ↓
Route 53
 ↓
CloudFront
 ↓
ALB
 ↓
Private EC2/ECS/EKS
 ↓
RDS/Aurora
```

### 20. Administrator Flow

Administrative access is:

```text
Administrator
 ↓
Systems Manager
 ↓
Private EC2
```

No public IP or Internet-facing SSH is required.

### 21. Private EC2 → Internet

For a dependency that genuinely requires public Internet access:

```text
Private EC2
 ↓
Private Route Table
 ↓
NAT Gateway
 ↓
Public Route Table
 ↓
Internet Gateway
 ↓
Internet
```

### 22. Private EC2 → S3

Prefer:

```text
Private EC2
 ↓
S3 VPC Endpoint
 ↓
S3
```

where appropriate.

### 23. VPC-A → VPC-B

For a small number of VPCs, VPC Peering can provide private connectivity.

For the multi-account/multi-VPC architecture described here, Transit Gateway is more scalable:

```text
VPC-A
 ↓
TGW Attachment
 ↓
TGW Route Table
 ↓
TGW Attachment
 ↓
VPC-B
```

### 24. Security Model

The overall trust model becomes:

```text
Internet
   ↓
CloudFront / ALB
   ↓
Private Application
   ↓
Private Database
```

with:

- No public IPs on application servers.
- No public database access.
- Least-privilege security groups.
- NACLs as an additional subnet-level control.
- IAM roles rather than long-lived AWS credentials.
- KMS-based encryption where required.
- CloudWatch for monitoring.
- CloudTrail for AWS API auditing.

This architecture gives you **multi-AZ availability, private application/database tiers, controlled outbound access, secure private-server administration, container support through ECS/EKS/ECR, Lambda-to-private-database connectivity, and controlled multi-VPC networking through Transit Gateway.**

---

# Interview Answer Pattern

For networking scenarios, structure your answer like this:

```text
1. Identify the source
2. Identify the destination
3. Identify the subnet
4. Check route table
5. Check IGW/NAT/TGW/Peering
6. Check Security Group
7. Check NACL
8. Check DNS
9. Check application port
10. Check OS/application
```

For example:

```text
Private EC2
    ↓
Route Table
    ↓
NAT Gateway
    ↓
Internet Gateway
    ↓
Internet
```

If the connection fails, don't immediately blame the Security Group.

Trace the packet **hop by hop**.

---

# Core AWS Networking Flows to Memorize

## Public EC2 → Internet

```text
EC2
 ↓
Public Subnet
 ↓
Route Table
 ↓
Internet Gateway
 ↓
Internet
```

## Private EC2 → Internet

```text
EC2
 ↓
Private Subnet
 ↓
Route Table
 ↓
NAT Gateway
 ↓
Public Subnet
 ↓
Internet Gateway
 ↓
Internet
```

## Internet → Private EC2 Application

```text
Internet
 ↓
ALB
 ↓
Private EC2
```

The Internet does **not** directly connect to the private EC2.

## Private EC2 → RDS

```text
EC2
 ↓
VPC local routing
 ↓
RDS
```

Security groups determine whether the database connection is permitted.

## VPC-A → VPC-B

```text
VPC-A
 ↓
VPC Peering
 ↓
VPC-B
```

or:

```text
VPC-A
 ↓
TGW Attachment
 ↓
Transit Gateway
 ↓
TGW Attachment
 ↓
VPC-B
```

## Administrator → Private EC2

Preferred pattern:

```text
Administrator
 ↓
Systems Manager Session Manager
 ↓
Private EC2
```

## ECS/EKS → ECR

```text
ECS/EKS
 ↓
VPC networking
 ↓
ECR
```

The workload needs both **network connectivity** and the appropriate **IAM permissions**.

## Lambda → Private RDS

```text
API Gateway
 ↓
Lambda
 ↓
VPC
 ↓
Private RDS/Aurora
```

The Lambda networking configuration and security groups must permit the database connection.

---

# Final Practice Rule

When answering these in an interview, avoid answers such as:

> "I will check the security group."

Instead say:

> "I will trace the traffic path from source to destination. First I'll verify DNS and the destination IP, then the source subnet's route table, the required routing component such as IGW/NAT/Peering/TGW, security groups on both sides, subnet NACLs, and finally the target application's listening port and OS firewall."

That demonstrates **AWS networking troubleshooting**, rather than simply memorizing AWS service definitions.
