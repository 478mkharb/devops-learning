# AMI

### Q1. What is an AMI?

**Answer:** An AMI (Amazon Machine Image) is a template used to launch EC2 instances. It contains the operating-system image and references to the block-device mappings required when the instance is launched.

---

### Q2. What does an AMI contain?

**Answer:** An AMI contains the software configuration needed to launch an EC2 instance, including the root device image, block-device mappings, architecture, virtualization information, and launch permissions. For EBS-backed AMIs, the image references EBS snapshots.

---

### Q3. What is the difference between an AMI and an EC2 instance?

**Answer:** An AMI is a reusable template; an EC2 instance is a running or stopped compute resource launched from a template. One AMI can be used to launch many instances.

---

### Q4. What is an EBS-backed AMI?

**Answer:** An EBS-backed AMI uses an EBS snapshot for its root device. Instances launched from it use EBS for the root volume and can normally be stopped and started while retaining that storage.

---

### Q5. What is an instance-store-backed AMI?

**Answer:** An instance-store-backed AMI uses instance store for the root device. The root storage is ephemeral and does not provide the persistence and stop/start behavior associated with EBS-backed instances.

---

### Q6. How do you create an AMI from an EC2 instance?

**Answer:** You can create an AMI from an EC2 instance using the EC2 console or CreateImage API/CLI. AWS captures the required EBS volume state as snapshots and creates an AMI that can be used to launch new instances.

---

### Q7. What happens to EBS snapshots when an AMI is created?

**Answer:** For an EBS-backed AMI, AWS creates snapshots of the EBS volumes included in the AMI's block-device mapping. The AMI references those snapshots when launching instances.

---

### Q8. Can AMIs be copied across Regions?

**Answer:** Yes. EBS-backed AMIs can be copied to another AWS Region. AWS creates copies of the required snapshots in the destination Region and registers the copied AMI there.

---

### Q9. Can AMIs be shared with another AWS account?

**Answer:** Yes, an AMI owner can share an AMI with specific AWS accounts by changing its launch permissions, subject to the AMI and snapshot configuration. Sharing an AMI does not transfer ownership.

---

### Q10. What is an AMI launch permission?

**Answer:** AMI launch permissions determine which AWS accounts can use an AMI to launch instances. An AMI can be private, shared with specific accounts, or public when appropriate.

---

### Q11. Why use a golden AMI?

**Answer:** A golden AMI provides a standardized, tested baseline containing the operating system, security updates, agents, and application dependencies. It makes instance launches faster and more consistent.

---

### Q12. How does AMI versioning help deployments?

**Answer:** Versioned AMIs let you identify exactly which image was used for a release. A deployment can launch a new image while retaining the previous image for rollback, improving consistency and auditability.

---

### Q13. What is the relationship between AMI, snapshot, and EBS volume?

**Answer:** An EBS volume is the live block-storage device attached to an instance. A snapshot is a point-in-time backup of that volume. An EBS-backed AMI uses snapshots and block-device mappings as the blueprint for launching new instances.

---

### Q14. What should be removed or generalized before creating a reusable image?

**Answer:** Remove instance-specific or sensitive data such as credentials, private keys, temporary files, logs, and environment-specific configuration. Generalize identifiers where required by the operating system, and leave reusable baseline configuration in the image.

---

### Q15. How can EC2 Image Builder automate image creation?

**Answer:** EC2 Image Builder uses pipelines, recipes, components, and tests to automate image creation. It can install updates and software, run validation tests, version the resulting image, and distribute the AMI to selected Regions or accounts.

---
