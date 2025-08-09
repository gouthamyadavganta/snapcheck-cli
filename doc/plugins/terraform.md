# Terraform Plugin

The Terraform plugin audits **infrastructure state and posture** using Terraform state as the source of truth.  
It detects **drift, public exposure, IAM risks, stale resources, and cost signals**, and emits structured findings for correlation.

---

## Capabilities (v1.0)

- **State ingestion:** Local or remote `.tfstate` (S3/HTTP backends supported).
- **Drift detection:** Cross-check selected resources against live cloud APIs.
- **Exposure checks:** Public S3 buckets, wide-open Security Groups, public ELBs.
- **IAM risk checks:** Admin-equivalent roles, wildcard `Action`/`Resource`, inline policies.
- **Hygiene:** Stale/orphaned resources, unreferenced outputs, ancient state timestamps.
- **Cost signals:** Heuristic cost hints from instance families/sizes (best-effort).
- **Trends:** Resource count deltas vs last run (surface sudden infra changes).

---

## Prerequisites

- Valid `.tfstate` file (local path or remote backend URL/S3).
- Optional cloud credentials to enable drift and exposure cross-checks:
  - **AWS:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, (optional) `AWS_SESSION_TOKEN`, `AWS_REGION`
- No Terraform CLI required at runtime.

> The plugin **does not** apply plans or modify infrastructure.

---

## Profile Configuration

```yaml
modules:
  - terraform

terraform:
  # Path to local tfstate OR remote object (S3/HTTP)
  state_path: ./state.tfstate
  # OR:
  # state_path: s3://my-tfstate-bucket/prod/terraform.tfstate
  # state_path: https://example.com/prod/terraform.tfstate

  # Enable/disable specific checks (all true by default)
  checks:
    drift: true
    exposure: true
    iam: true
    hygiene: true
    cost: true
    trends: true

  # Performance / safety
  limits:
    max_resources: 20000        # hard stop to avoid runaway audits
    drift_sample_percent: 100   # 1–100; reduce for very large estates

  # AWS-specific options (if drift/exposure is enabled)
  aws:
    region: us-east-1
Environment variables (optional for drift/exposure):

bash
Copy
Edit
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=... # if applicable
export AWS_REGION=us-east-1
Input Sources
Local File
terraform.state_path: ./path/to/terraform.tfstate

S3 Backend
terraform.state_path: s3://<bucket>/<key>

Uses standard AWS SDK resolution for credentials.

HTTP(S)
terraform.state_path: https://example.com/foo.tfstate

Expects plain JSON state; authentication handled by environment if your setup fetches it before passing to the plugin (e.g., presigned URL).

Checks (Detailed)
1) Drift Detection
What: Compare resources in .tfstate against live cloud state.
How:

For AWS: Uses boto3 calls (EC2, S3, ELB, IAM subsets).

Compares attributes like existence, tags, exposure flags, instance size/type, listener ports.

Emits critical/high when infra diverges materially (missing resource, wrong type, public when state says private).

Examples surfaced:

i-abc123 in state but not found in EC2.

S3 bucket policy allows public read; state has no such intent.

ELB listener exists with unexpected open port.

Notes:

Exact properties diff vary by resource type. Large estates can use drift_sample_percent to sample.

2) Public Exposure
What: Identify publicly reachable resources that should be private.
Signals:

S3: BlockPublicAcls/PublicPolicy disabled or bucket policy with Principal:"*" and Action:"s3:GetObject".

Security Groups: Inbound rules with 0.0.0.0/0 on sensitive ports (22, 3389, 80/443 flagged with lower severity unless combined with other signals).

Load Balancers: Public-facing with insecure listeners (plaintext where TLS expected).

Severity:

Public S3 with object read: critical

SG 0.0.0.0/0 on 22/3389: high

Public HTTP only (no TLS): medium

3) IAM Risk
What: Surface over-privileged identities or wildcard policies.
Signals:

Action: "*", Resource: "*" in inline or attached policies.

AWS managed AdministratorAccess attached to roles/users/groups.

Wildcard resource on sensitive actions (e.g., kms:*, iam:PassRole, sts:AssumeRole without condition).

Severity:

Admin-equivalent: critical

Wildcards on sensitive namespaces: high

Benign wildcards with scope boundaries: medium/low

4) Hygiene
What: Cleanliness/maintainability issues reducing reliability.

Ancient state (e.g., serial unchanged for long periods; last modified >> 90 days).

Unreferenced outputs in state.

Dangling resources suspected (referenced in outputs but not deployed).

Module bloat (unused modules left in state).

Severity: mostly low/medium, escalates if combined with drift.

5) Cost Hints
What: Best-effort heuristics to highlight potential spend areas.

Count by instance family/size (e.g., many m5.4xlarge).

Unused/burstable anomalies (e.g., too many t2 with sustained CPU).

Redundant load balancers.

These are signals, not billing truth. The dedicated Cost plugin is authoritative for spend.

6) Trends
What: Compare resource counts and key metrics with the previous run.

Resource deltas (per type).

IAM policy count drift.

Exposure count trend.

Output Schema
All plugin findings are converted into AuditResult entries.

json
Copy
Edit
{
  "plugin": "terraform",
  "status": "high",                // critical|high|medium|low|ok|error
  "messages": [
    "S3 bucket prod-artifacts is public (policy allows s3:GetObject to *)"
  ],
  "metadata": {
    "resource_id": "arn:aws:s3:::prod-artifacts",
    "category": "exposure",
    "evidence": {
      "policy_snippet": "{...}",
      "public": true
    }
  }
}
Status mapping:

critical — Immediate action required (public data, admin perms, hard drift)

high — High-risk misconfig (wide-open SG on admin ports, wildcard IAM on sensitive actions)

medium — Not ideal; address soon (outdated charts, non-TLS listeners)

low — Hygiene improvements (stale outputs, old state)

error — Plugin execution error (non-fatal to overall run)

ok — Explicit pass (optional)

Example Findings
Public S3
pgsql
Copy
Edit
[CRITICAL][terraform] S3 bucket prod-artifacts is public (policy grants s3:GetObject to *)
SG Open to World
csharp
Copy
Edit
[HIGH][terraform] Security group sg-1234 inbound 22/tcp from 0.0.0.0/0
Admin Policy
pgsql
Copy
Edit
[CRITICAL][terraform] Role 'ci-runner' has AdministratorAccess policy attached
Drift: Missing Resource
css
Copy
Edit
[HIGH][terraform] EC2 i-0abc123 referenced in state not found in AWS (possible manual deletion)
Hygiene: Unreferenced Output
csharp
Copy
Edit
[LOW][terraform] Output 'db_endpoint' is not referenced by any module
Execution Flow
Load state (local/S3/HTTP).

Parse resources (types, attributes, outputs).

Run checks (exposure, IAM, hygiene, cost hints).

Optional drift (live AWS describes to verify existence/exposure).

Summarize and emit AuditResult list.

Trends (compare with last run’s snapshot).

Performance & Limits
Designed for large states (10k–20k resources) with guardrails:

limits.max_resources to abort early if unexpectedly large

drift_sample_percent to sample verifies for extremely large fleets

Uses batched AWS API calls where available.

Retries with exponential backoff for rate limits.

Security Notes
Read-only AWS permissions recommended (describe/list/get).

No Terraform credentials or backends are modified.

State files are treated as sensitive artifacts—avoid publishing raw state.

Suggested AWS policy (minimal):

json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {"Effect":"Allow","Action":["ce:Get*","ce:List*"],"Resource":"*"},
    {"Effect":"Allow","Action":["ec2:Describe*","elasticloadbalancing:Describe*","iam:Get*","iam:List*","s3:Get*","s3:List*"],"Resource":"*"}
  ]
}
(Include CE only if you want cost cross-checks here; otherwise omit.)

Troubleshooting
Symptom	Cause	Resolution
state file not found	Bad state_path	Fix path/URL; for S3 ensure bucket/key exist
AccessDenied (S3)	Missing S3 permissions	Grant s3:GetObject on the tfstate key
Drift check skipped	No AWS creds	Export AWS env vars or disable checks.drift
Many Plugin Error entries	Malformed state	Rebuild state from Terraform; validate JSON
False-positive exposure	Intentional public bucket	Add exception notes; confirm with Security; tune profile if needed

Examples
Minimal Local State Audit
bash
Copy
Edit
snapcheck run audit --modules terraform --output terminal \
  --profile profiles/prod.yaml
Profile:

yaml
Copy
Edit
terraform:
  state_path: ./infra/terraform.tfstate
S3 State with Drift Sampling
yaml
Copy
Edit
terraform:
  state_path: s3://company-tfstate/prod/terraform.tfstate
  checks:
    drift: true
  limits:
    drift_sample_percent: 30
Related Docs
CLI: snapcheck run

Architecture

Security

Developer: Correlation Engine