# Cost Plugin (AWS Cost Explorer)

The Cost plugin integrates with **AWS Cost Explorer** to retrieve **real monthly service costs** and cross-checks them against the Terraform-managed resource inventory.  
It helps detect **spend anomalies**, **unexpected services**, and **cost trends**.

---

## Capabilities (v1.0)

- Queries AWS Cost Explorer for the last full month and current month-to-date.
- Breaks down spend by:
  - Service (e.g., AmazonEC2, AmazonS3)
  - Linked account (if in an AWS Organization)
- Detects services **not referenced in Terraform state**.
- Flags sudden cost spikes compared to historical averages.
- Provides **total cost** and **per-service cost** in reports.

---

## Prerequisites

- AWS Cost Explorer enabled in the AWS account.
- IAM permissions:
  - `ce:GetCostAndUsage`
  - `ce:GetDimensionValues` (optional, for account mapping)
- Optional: Terraform state path configured in `terraform` module for cross-check.

Example minimal AWS policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {"Effect": "Allow", "Action": ["ce:GetCostAndUsage", "ce:GetDimensionValues"], "Resource": "*"}
  ]
}
Profile Configuration
yaml
Copy
Edit
modules:
  - cost

cost:
  aws_region: us-east-1
  granularity: MONTHLY         # DAILY|MONTHLY
  lookback_months: 3
  anomaly_threshold_percent: 25
  crosscheck_with_terraform: true
Checks (Detailed)
1) Cost Breakdown
Signals:

Total spend for the period.

Per-service spend table.

Linked account breakdown if applicable.

Severity: Informational unless anomalies detected.

2) Anomaly Detection
Signals:

Service cost deviates > anomaly_threshold_percent from average of lookback period.

Severity:

Spike > 50%: high

Spike > threshold but < 50%: medium

3) Unmanaged Service Spend
Signals:

Service with > $X spend not represented in Terraform-managed resources.

Severity:

$500/month unmanaged: high

$100/month unmanaged: medium

Output Schema
json
Copy
Edit
{
  "plugin": "cost",
  "status": "medium",
  "messages": [
    "AmazonS3 spend increased 35% vs average ($230 → $310)"
  ],
  "metadata": {
    "service": "AmazonS3",
    "current": 310.0,
    "average": 230.0,
    "delta_percent": 35
  }
}
Example Findings
Spend Spike
scss
Copy
Edit
[HIGH][cost] AmazonEC2 spend increased 65% vs average ($1,500 → $2,475)
Unmanaged Service
scss
Copy
Edit
[MEDIUM][cost] AWS Lambda ($400/month) not found in Terraform state resources
Execution Flow
Authenticate with AWS credentials (from env or profile config).

Query Cost Explorer for the configured time period and granularity.

Aggregate by service (and linked account if available).

Calculate anomalies by comparing against lookback average.

If enabled, cross-check with Terraform state resource types.

Emit AuditResult list.

Performance & Limits
Cost Explorer data is delayed ~24 hours.

Lookback > 12 months increases query time.

Linked account breakdown requires AWS Organizations setup.

Security Notes
Use read-only IAM policies for CE data.

No sensitive billing account information is stored in SnapCheck history — only aggregated spend values.

Troubleshooting
Symptom	Cause	Resolution
AccessDenied	Missing ce:GetCostAndUsage permission	Add to IAM policy
All costs show $0	Cost Explorer not enabled	Enable in AWS console
Cross-check always empty	Terraform state not linked or module disabled	Enable crosscheck_with_terraform in profile

Examples
Monthly Cost Summary
bash
Copy
Edit
snapcheck run audit --modules cost --profile profiles/prod.yaml
Daily Cost Tracking
yaml
Copy
Edit
cost:
  aws_region: us-east-1
  granularity: DAILY
  lookback_months: 1
Related Docs
CLI: snapcheck run

Architecture

Terraform Plugin