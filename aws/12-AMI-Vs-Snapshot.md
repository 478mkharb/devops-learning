# AMI vs Snapshot (AWS EC2)

Professional reference comparing **Amazon Machine Images (AMI)** and **EBS Snapshots**.

---

## AMI vs Snapshot

| Feature            | AMI (Amazon Machine Image)                                      | Snapshot (EBS Snapshot)               |
| ------------------ | --------------------------------------------------------------- | ------------------------------------- |
| What it is         | EC2 image/template used to launch instances                     | Point-in-time backup of an EBS volume |
| Used for           | Launching EC2 instances                                         | Restoring or creating EBS volumes     |
| Contains           | OS + installed software + configuration + block device mappings | Raw disk data only                    |
| Scope              | Region-specific                                                 | Region-specific                       |
| Bootable           | Yes                                                             | No                                    |
| Created from       | One or more EBS snapshots                                       | Existing EBS volume                   |
| Auto Scaling usage | Used in Launch Templates / Auto Scaling Groups                  | Not used directly                     |

---

## Relationship Between AMI and Snapshot

```
EBS Volume
   |
   v
Snapshot (backup of volume)
   |
   v
AMI (built using snapshot)
   |
   v
Launch EC2 Instance
```

---

## DevOps Use Cases

| Scenario                     | Recommended Resource |
| ---------------------------- | -------------------- |
| Backup EC2 disk              | Snapshot             |
| Clone EC2 server             | AMI                  |
| Auto Scaling launch template | AMI                  |
| Disaster recovery            | Snapshot             |
| Golden server image          | AMI                  |

---

## Quick Interview Explanation

**AMI** is a template used to launch EC2 instances and contains the operating system, software, and configuration.

**Snapshot** is a point-in-time backup of an EBS volume that stores only the disk data.
