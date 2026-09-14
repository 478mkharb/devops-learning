# Amazon Machine Images (AMI)


## 1. What is an AMI?

**Answer:** An AMI (Amazon Machine Image) is a template used to launch EC2 instances. It contains the operating-system image, software configuration, and block-device mappings required to launch an instance.

## 2. What does an AMI contain?

**Answer:** An AMI can contain or reference:

- Operating-system image
- Installed software and configuration
- Root device information
- Block-device mappings
- Architecture, such as `x86_64` or `arm64`
- Virtualization information
- Launch permissions
- EBS snapshots for EBS-backed images

---

## 3. Classification of AMIs

AMIs can be classified by **ownership/source**, **storage backing**, and **visibility**.

### 3.1 Classification by ownership or source

| AMI classification | Meaning | Example |
|---|---|---|
| **AWS-managed AMI** | Image provided and maintained by AWS | Amazon Linux AMI |
| **AWS Marketplace AMI** | Image published by a third-party vendor through AWS Marketplace | Commercial Linux, security, or database appliance image |
| **Community AMI** | AMI shared publicly by another AWS user or organization | Publicly shared application image |
| **Custom AMI** | AMI created by an individual or organization for its own requirements | Company-standard Ubuntu image |
| **Golden AMI** | Tested, hardened, versioned custom AMI used as an approved baseline | Production application golden image |

> **Important:** A golden AMI is a purpose and operating model, not a separate AWS technical storage type. It is usually a custom AMI.

### 3.2 Classification by storage backing

| Type | Description | Important behavior |
|---|---|---|
| **EBS-backed AMI** | Root device is backed by an EBS snapshot | Supports the normal EBS-backed stop/start model |
| **Instance-store-backed AMI** | Root device is backed by instance store | Root storage is ephemeral and is not preserved like EBS storage |

**Practical note:** Most modern EC2 workloads use EBS-backed AMIs. Instance-store-backed AMIs are uncommon and depend on supported instance types and legacy workflows.

### 3.3 Classification by visibility

| Visibility | Meaning |
|---|---|
| **Private AMI** | Available only to the owning AWS account by default |
| **Shared AMI** | Launch permission granted to selected AWS accounts |
| **Public AMI** | Available for public discovery and use, subject to its permissions and configuration |

---

## 4. What is the relationship between an AMI, snapshot, and EBS volume?

| Component | Meaning |
|---|---|
| **EBS volume** | Live block-storage device attached to an EC2 instance |
| **Snapshot** | Point-in-time backup of an EBS volume |
| **AMI** | Launch blueprint that references snapshots and block-device mappings |

### Launch flow

```text
EBS volume
    |
    | Create snapshot
    v
EBS snapshot
    |
    | Used by AMI
    v
EBS-backed AMI
    |
    | Launch
    v
New EC2 instance with new EBS volume
```
---

## 5. What happens to EBS snapshots when an AMI is created?

**Answer:** For an EBS-backed AMI, AWS creates or uses snapshots for the EBS volumes included in the AMI's block-device mappings. The AMI references those snapshots when launching new instances.

---

## 6. How do you create an AMI from an EC2 instance?

**Answer:** An AMI can be created from an EC2 instance through the EC2 console or the `CreateImage` API/CLI operation.

Typical process:

1. Prepare the instance.
2. Remove credentials, secrets, temporary files, and unnecessary logs.
3. Stop the instance if application consistency requires it.
4. Create the AMI.
5. Wait until the AMI becomes available.
6. Test launching an instance from the new AMI.
7. Record the AMI ID and version.

### AWS CLI example

```bash
aws ec2 create-image \
  --instance-id i-0123456789abcdef0 \
  --name "otms-notification-golden-v1" \
  --description "Version 1 golden AMI for notification service" \
  --no-reboot
```

> `--no-reboot` avoids an automatic reboot, but it can reduce filesystem and application consistency. Use it only when the workload can safely tolerate that approach.

---

## 7. What is the difference between deregistering and deleting an AMI?

**Answer:**

- **Deregistering an AMI** removes the AMI registration so it cannot be used for new launches through that AMI.
- **Deleting snapshots** removes the EBS snapshots referenced by the AMI.
- Deregistering an AMI does not automatically mean that every related snapshot has been deleted.

> Delete snapshots only after confirming that they are not required by another AMI, backup process, or recovery plan.

## 8. Can an AMI be deleted while instances launched from it are running?

**Answer:** Yes. Deregistering an AMI does not terminate existing instances launched from it. Those instances continue running, but the deregistered AMI cannot be used for new launches.
---

## 9. Can AMIs be copied across Regions?

**Answer:** Yes. EBS-backed AMIs can be copied to another AWS Region. AWS copies the required snapshots to the destination Region and creates a new AMI there.

---

## 10. Does copying an AMI preserve the AMI ID?

**Answer:** No. The copied AMI receives a different AMI ID in the destination Region.

---

## 11. Can AMIs be shared with another AWS account?

**Answer:** Yes. An AMI owner can grant launch permissions to specific AWS accounts. Sharing does not transfer ownership.

---

## 12. What is an AMI launch permission?

**Answer:** AMI launch permissions determine which AWS accounts can use an AMI to launch instances.

An AMI can be:

- Private
- Shared with selected accounts
- Public, when appropriate

---

## 13. What is a golden AMI?

**Answer:** A golden AMI is a standardized, tested, and approved image used as the baseline for launching instances. It commonly contains:

- Approved operating system
- Security updates
- Monitoring and management agents
- Required runtime packages
- Application dependencies
- Standard configuration
- Security hardening
- Validation results

---

## 14. Why use a golden AMI?

**Answer:** Golden AMIs provide:

- Faster instance provisioning
- Consistent environments
- Reduced configuration drift
- Repeatable deployments
- Easier troubleshooting
- Better auditability
- Safer rollback to a previous image version

---

## 15. How does AMI versioning help deployments?

**Answer:** Versioned AMIs identify the exact image used for a release. A new image can be tested and deployed while the previous image is retained for rollback.

### Recommended naming pattern

```text
<application>-<environment>-<purpose>-v<version>
```

Example:

```text
otms-dev-notification-golden-v1
```

### Recommended metadata

Record:

- AMI ID
- AMI name
- Version
- Creation date
- Source instance or build pipeline
- Operating-system version
- Application version
- Security patch level
- Owner
- Test status
- Target Regions
- Deprecation date

---

## 16. What should be removed or generalized before creating a reusable image?

Remove or clean:

- Passwords and credentials
- Private keys
- API tokens
- SSH host keys when required by the operating system
- Temporary files
- Shell history containing secrets
- Application logs
- Environment-specific configuration
- Instance-specific identifiers
- Unnecessary packages
- Cached credentials

Keep:

- Approved operating-system configuration
- Required agents
- Required runtime packages
- Baseline security settings
- Reusable application dependencies

---

## 17. Why should secrets not be baked into an AMI?

**Answer:** Every instance launched from the AMI may inherit the baked-in secret. This increases the impact of image sharing, copying, accidental exposure, and compromise.

Use a secure runtime configuration mechanism instead of embedding secrets in the image.

---

## 18. How can EC2 Image Builder automate image creation?

**Answer:** EC2 Image Builder automates image creation through:

- Image pipelines
- Image recipes
- Components
- Build steps
- Test steps
- Versioning
- Distribution settings

It can install updates and software, apply configuration, run validation tests, and distribute the resulting AMI to selected Regions or accounts.

---

## 19. What are the main EC2 Image Builder concepts?

| Concept | Meaning |
|---|---|
| **Image pipeline** | Automated workflow for building and testing images |
| **Image recipe** | Defines the base image and components |
| **Component** | Build or test instructions |
| **Build phase** | Installs and configures software |
| **Test phase** | Validates the resulting image |
| **Distribution configuration** | Defines where the image is copied |
| **AMI version** | Identifies a particular image build |

### Example workflow

```text
Base AMI
   |
   v
Install security updates
   |
   v
Install agents and dependencies
   |
   v
Apply hardening
   |
   v
Run tests
   |
   v
Create versioned AMI
   |
   v
Distribute to selected Regions/accounts
```

---

## 20. Common AMI Interview Scenarios

### Q1. You created an AMI but the new instance does not have the latest application changes. Why?

**Answer:** Possible causes include:

- The application changes were not present before image creation.
- The wrong source instance was used.
- The AMI was created before the changes were completed.
- The launch process used an older AMI ID.
- The application data was stored outside the captured root volume.

### Q2. You deregistered an AMI. Are existing instances terminated?

**Answer:** No. Deregistering an AMI does not terminate instances already launched from it.

### Q3. You shared an AMI, but another account cannot launch it. What should you check?

Check:

- AMI launch permissions
- Snapshot permissions for the backing snapshots
- Region of the AMI
- Architecture and instance compatibility
- Encryption and KMS key permissions, if applicable
- Whether the AMI is still available

### Q4. Why might an AMI launch fail after copying it to another Region?

Possible causes include:

- Missing or incorrect snapshot permissions
- KMS key access problems for encrypted snapshots
- Unsupported architecture or instance type
- Region-specific resource references
- Missing launch configuration dependencies

---

## 21. Quick Revision

| Question | Short answer |
|---|---|
| What is an AMI? | A reusable image blueprint for launching EC2 instances |
| Main AMI source types? | AWS-managed, Marketplace, Community, Custom, Golden |
| Main backing types? | EBS-backed and instance-store-backed |
| Main visibility types? | Private, shared, and public |
| What does an EBS-backed AMI reference? | EBS snapshots and block-device mappings |
| Can AMIs be copied across Regions? | Yes |
| Does a copied AMI keep the same ID? | No |
| Does sharing transfer ownership? | No |
| Does deregistering terminate instances? | No |
| Why use a golden AMI? | Consistency, speed, security, and rollback |
| What automates image creation? | EC2 Image Builder |

---

## 22. Interview Checkpoints

- Define an AMI in one sentence.
- Explain what an AMI contains.
- Classify AMIs by source, backing, and visibility.
- Compare EBS-backed and instance-store-backed AMIs.
- Explain the AMI–snapshot–EBS volume relationship.
- Describe how to create an AMI.
- Explain `--no-reboot` and its consistency trade-off.
- Explain AMI sharing and launch permissions.
- Explain cross-Region AMI copying.
- Differentiate deregistering an AMI from deleting snapshots.
- Explain golden AMIs and versioning.
- Describe how EC2 Image Builder creates and tests AMIs.
