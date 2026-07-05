
# AWS Budget and Cost Anomaly Alerts

## What is AWS Budget?

AWS Budgets allows you to define **cost or usage limits** and sends alerts when your spending reaches predefined thresholds.

Example:

Monthly Budget = **$500**

Alerts

- 50% ($250)
- 80% ($400)
- 100% ($500)

---

## What is AWS Cost Anomaly Detection?

AWS Cost Anomaly Detection uses **Machine Learning (ML)** to detect unusual spending patterns automatically.

Example

Normal EC2 Cost

```text
₹500/day
```

Suddenly

```text
₹8,000/day
```

AWS automatically detects the abnormal increase and sends an alert.

---

## Benefits

- Prevent unexpected bills
- Detect unusual resource usage
- Monitor spending continuously
- Early notification of cost spikes

---

## Budget vs Cost Anomaly Detection

| AWS Budget | Cost Anomaly Detection |
|------------|------------------------|
| User defines budget | AWS uses Machine Learning |
| Threshold-based alerts | Detects unusual spending automatically |
| Predictable monitoring | Intelligent anomaly detection |
| Example: $1000/month | Example: EC2 cost suddenly increases 500% |

---

## DevOps Interview Answer

> AWS Budgets provides threshold-based alerts for planned spending, while AWS Cost Anomaly Detection uses machine learning to identify unexpected increases in AWS costs.
