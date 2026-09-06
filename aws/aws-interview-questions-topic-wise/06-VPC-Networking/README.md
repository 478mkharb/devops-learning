# VPC & AWS Networking

### Q1. What is a VPC?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, AWS services, or on-premises networks.

---

### Q2. What is a CIDR block?

**Answer:** A CIDR block defines an IP address range, such as 10.0.0.0/16. The prefix length determines how many addresses are included in the range.

---

### Q3. What is a subnet?

**Answer:** A subnet is an IP address range inside a VPC and is associated with one Availability Zone. A subnet is commonly called public when its route table has a route to an Internet Gateway.

---

### Q4. What is the difference between public and private subnets?

**Answer:** A public subnet has a route to an Internet Gateway and can host resources with appropriate public addressing. A private subnet does not have a direct route to an Internet Gateway; outbound Internet access, if required, is normally provided through a NAT Gateway.

---

### Q5. What is a route table?

**Answer:** A route table contains destination-to-target rules that determine where VPC traffic is sent. Subnets are associated with route tables, and the most specific matching route is selected.

---

### Q6. What is a route?

**Answer:** A VPC route maps a destination CIDR to a target such as an Internet Gateway, NAT Gateway, VPC peering connection, Transit Gateway, or VPN-related target.

---

### Q7. What is an Internet Gateway?

**Answer:** An Internet Gateway is a VPC component that provides a path between the VPC and the Internet when routing and public addressing are configured appropriately. It is attached to the VPC.

---

### Q8. What is a NAT Gateway?

**Answer:** A NAT Gateway allows resources in private subnets to initiate outbound connections to the Internet without allowing unsolicited inbound Internet connections. It is normally deployed in a public subnet with a route to an Internet Gateway.

---

### Q9. Why does a private subnet need a NAT Gateway for outbound Internet access?

**Answer:** A private subnet uses a NAT Gateway when its resources need to initiate Internet-bound connections but should not have a direct Internet Gateway route. The private route table sends outbound traffic to the NAT Gateway, which performs address translation and sends it through an Internet Gateway.

---

### Q10. Can a NAT Gateway accept unsolicited inbound Internet traffic?

**Answer:** No. A NAT Gateway allows return traffic for connections initiated by private resources, but it does not provide unsolicited inbound access from the Internet to those private resources.

---

### Q11. What is VPC peering?

**Answer:** VPC peering provides private IP connectivity between two VPCs. It is non-transitive, so traffic cannot automatically pass from one peered VPC through another peered VPC.

---

### Q12. What is Transit Gateway?

**Answer:** AWS Transit Gateway is a central network hub for connecting multiple VPCs and on-premises networks. It simplifies routing compared with maintaining many individual VPC peering connections.

---

### Q13. When is Transit Gateway preferable to many VPC peerings?

**Answer:** Transit Gateway is preferable when many VPCs and/or on-premises networks need connectivity. Instead of building and maintaining a large mesh of individual peerings, each network can connect to the central Transit Gateway and use its route tables for controlled communication.

---

### Q14. What is a VPC endpoint?

**Answer:** A VPC endpoint provides private connectivity from a VPC to supported AWS services or endpoint services without requiring the service path to traverse the public Internet.

---

### Q15. What is a gateway endpoint?

**Answer:** A gateway endpoint provides private connectivity from VPC route tables to supported services such as Amazon S3 and DynamoDB. It does not create an ENI in the subnet.

---

### Q16. What is an interface endpoint?

**Answer:** An interface endpoint creates one or more elastic network interfaces in selected subnets and uses AWS PrivateLink to provide private connectivity to supported services. Security groups control traffic to the endpoint ENIs.

---

### Q17. What is AWS PrivateLink?

**Answer:** AWS PrivateLink provides private connectivity to supported AWS services, endpoint services, and SaaS applications through interface endpoints. The traffic does not require an Internet Gateway or public IP path.

---

### Q18. When should you use an S3/DynamoDB gateway endpoint?

**Answer:** Use a gateway endpoint when resources in a VPC need private connectivity to S3 or DynamoDB without sending traffic through a NAT Gateway or public Internet path. The endpoint is associated with route tables and has no hourly endpoint charge for these gateway endpoints.

---

### Q19. What role does Private DNS play with interface endpoints?

**Answer:** Private DNS for an interface endpoint lets applications continue using the normal AWS service hostname while DNS resolves that hostname to the private endpoint IP addresses inside the VPC. This avoids changing application URLs for private access.

---

### Q20. What is a security group?

**Answer:** A security group is a stateful virtual firewall attached to an ENI. It controls allowed inbound and outbound traffic, and return traffic for an allowed connection is automatically permitted.

---

### Q21. What is a network ACL?

**Answer:** A network ACL is a stateless subnet-level traffic filter. It evaluates numbered inbound and outbound rules, so return traffic must be explicitly allowed in the opposite direction.

---

### Q22. Compare security groups and NACLs.

**Answer:** Security groups are stateful firewalls attached to network interfaces and support allow rules. NACLs are stateless filters applied at the subnet boundary and support ordered allow and deny rules. A security group automatically permits response traffic for an allowed connection, while a NACL must explicitly allow both directions.

---

### Q23. Why are security groups stateful?

**Answer:** A security group is stateful because AWS tracks the connection state. If traffic is allowed in one direction, the corresponding response traffic is automatically permitted without requiring a separate reverse-direction rule.

---

### Q24. Why are NACLs stateless?

**Answer:** NACLs are stateless because each direction is evaluated independently. If inbound traffic is allowed, the corresponding return traffic must also be explicitly allowed by an outbound rule, and vice versa.

---

### Q25. What is VPC Flow Logs?

**Answer:** VPC Flow Logs capture metadata about network traffic to and from VPC network interfaces. They are useful for troubleshooting connectivity and investigating security events.

---

### Q26. What is a VPN connection?

**Answer:** An AWS Site-to-Site VPN provides encrypted IPsec connectivity between a VPC and an on-premises or compatible remote network. It is commonly used for hybrid connectivity.

---

### Q27. What is Direct Connect?

**Answer:** AWS Direct Connect provides a dedicated network connection from a customer network to AWS. It can provide more predictable network performance than Internet-based connectivity.

---

### Q28. What is a route propagation?

**Answer:** Route propagation allows routes learned through a virtual private gateway or supported connectivity to be automatically added to selected VPC route tables when propagation is enabled.

---

### Q29. What is a route table association?

**Answer:** A route table association links a subnet with a route table. A subnet uses the routes in its associated table for traffic leaving the subnet.

---

### Q30. What is a VPC DHCP option set?

**Answer:** A VPC DHCP option set defines network configuration values supplied to instances through DHCP, such as DNS servers, domain name, and related options.

---

### Q31. What is a Network Firewall?

**Answer:** AWS Network Firewall is a managed stateful network firewall that provides traffic inspection and filtering for VPC architectures. It can be deployed in centralized inspection patterns.

---
