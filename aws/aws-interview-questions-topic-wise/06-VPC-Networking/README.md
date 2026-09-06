# VPC & AWS Networking

## Interview Questions & Answers

### Q1. What is a VPC?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

---

### Q2. What is a CIDR block?

**Answer:** CIDR notation defines an IP address range, such as 10.0.0.0/16. A larger prefix length represents a smaller address range.

---

### Q3. What is a subnet?

**Answer:** A subnet is an IP address range inside a VPC and is associated with one Availability Zone. A subnet is considered public when its route table provides a path to an Internet Gateway; otherwise it is commonly private.

---

### Q4. What is the difference between public and private subnets?

**Answer:** A subnet is an IP address range inside a VPC and is associated with one Availability Zone. A subnet is considered public when its route table provides a path to an Internet Gateway; otherwise it is commonly private.

---

### Q5. What is a route table?

**Answer:** A route table contains destination-to-target rules that determine where VPC traffic is sent. Subnets are associated with route tables, and the most specific matching route is selected.

---

### Q6. What is a route?

**Answer:** Explain the component in the VPC traffic path, including subnet placement, route-table behavior, and relevant security controls. State whether the connectivity is Internet, private AWS, VPC-to-VPC, or hybrid.

---

### Q7. What is an Internet Gateway?

**Answer:** An Internet Gateway is a horizontally scaled VPC component that enables Internet connectivity for resources with appropriate public addressing and routes. It is attached to the VPC.

---

### Q8. What is a NAT Gateway?

**Answer:** A NAT Gateway allows resources in private subnets to initiate outbound connections to the Internet without accepting unsolicited inbound Internet connections. It is normally placed in a public subnet and requires a route to an Internet Gateway.

---

### Q9. Why does a private subnet need a NAT Gateway for outbound Internet access?

**Answer:** A subnet is an IP address range inside a VPC and is associated with one Availability Zone. A subnet is considered public when its route table provides a path to an Internet Gateway; otherwise it is commonly private.

---

### Q10. Can a NAT Gateway accept unsolicited inbound Internet traffic?

**Answer:** A NAT Gateway allows resources in private subnets to initiate outbound connections to the Internet without accepting unsolicited inbound Internet connections. It is normally placed in a public subnet and requires a route to an Internet Gateway.

---

### Q11. What is VPC peering?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

---

### Q12. What is Transit Gateway?

**Answer:** AWS Transit Gateway acts as a central network hub for connecting multiple VPCs and on-premises networks. It reduces the mesh of individual connections and supports centralized routing.

---

### Q13. When is Transit Gateway preferable to many VPC peerings?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

---

### Q14. What is a VPC endpoint?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

---

### Q15. What is a gateway endpoint?

**Answer:** A gateway VPC endpoint provides private connectivity from a VPC to supported AWS services such as S3 and DynamoDB through route-table entries. It does not use an ENI in the subnet.

---

### Q16. What is an interface endpoint?

**Answer:** An interface VPC endpoint creates elastic network interfaces in subnets and privately connects to supported AWS services through AWS PrivateLink. Security groups control traffic to the endpoint ENIs.

---

### Q17. What is AWS PrivateLink?

**Answer:** AWS PrivateLink provides private connectivity to supported AWS services, endpoint services, and SaaS applications without requiring Internet Gateway, NAT Gateway, or public IP connectivity for the service path.

---

### Q18. When should you use an S3/DynamoDB gateway endpoint?

**Answer:** A gateway VPC endpoint provides private connectivity from a VPC to supported AWS services such as S3 and DynamoDB through route-table entries. It does not use an ENI in the subnet.

---

### Q19. What role does Private DNS play with interface endpoints?

**Answer:** An interface VPC endpoint creates elastic network interfaces in subnets and privately connects to supported AWS services through AWS PrivateLink. Security groups control traffic to the endpoint ENIs.

---

### Q20. What is a security group?

**Answer:** A security group is a stateful virtual firewall attached to an ENI. It defines allowed inbound and outbound traffic. Return traffic for an allowed connection is automatically permitted.

---

### Q21. What is a network ACL?

**Answer:** A network ACL is a stateless subnet-level traffic filter. It evaluates numbered inbound and outbound rules, and return traffic must be explicitly allowed in the opposite direction.

---

### Q22. Compare security groups and NACLs.

**Answer:** A security group is a stateful virtual firewall attached to an ENI. It defines allowed inbound and outbound traffic. Return traffic for an allowed connection is automatically permitted.

---

### Q23. Why are security groups stateful?

**Answer:** A security group is a stateful virtual firewall attached to an ENI. It defines allowed inbound and outbound traffic. Return traffic for an allowed connection is automatically permitted.

---

### Q24. Why are NACLs stateless?

**Answer:** Explain the component in the VPC traffic path, including subnet placement, route-table behavior, and relevant security controls. State whether the connectivity is Internet, private AWS, VPC-to-VPC, or hybrid.

---

### Q25. What is VPC Flow Logs?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

---

### Q26. What is a VPN connection?

**Answer:** AWS Site-to-Site VPN creates encrypted IPsec connectivity between a VPC and an on-premises network or compatible remote network. It is commonly used for hybrid connectivity.

---

### Q27. What is Direct Connect?

**Answer:** AWS Direct Connect provides a dedicated network connection from a customer network to AWS. It can provide more predictable network performance than Internet-based connectivity.

---

### Q28. What is a route propagation?

**Answer:** Route propagation allows routes learned from a VPN or virtual private gateway to be automatically added to a route table when enabled.

---

### Q29. What is a route table association?

**Answer:** A route table contains destination-to-target rules that determine where VPC traffic is sent. Subnets are associated with route tables, and the most specific matching route is selected.

---

### Q30. What is a VPC DHCP option set?

**Answer:** A VPC is a logically isolated virtual network in AWS. It contains subnets, route tables, network interfaces, and security controls and can connect to the Internet, other VPCs, on-premises networks, or AWS services.

---

### Q31. What is a Network Firewall?

**Answer:** AWS Network Firewall is a managed, stateful network firewall for VPC traffic. It supports traffic inspection and filtering at the network layer and can be integrated into centralized inspection architectures.

---
