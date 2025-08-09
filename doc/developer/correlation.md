# Correlation Engine

The Correlation Engine is the **core intelligence layer** in SnapCheck.  
It consumes raw findings from all plugins and produces **storylines** that explain what broke, why it broke, and whether it’s new or recurring.

---

## Goals

- Connect related findings across multiple systems.
- Differentiate **root causes** from **symptoms**.
- Detect regressions compared to prior runs.
- Provide a single, readable incident narrative.

---

## How It Works

### 1) Input
- Each plugin returns a list of `AuditResult` objects:
  - `status` — severity (critical, high, medium, low, ok, error)
  - `messages` — list of human-readable findings
  - `metadata` — structured details (resource IDs, namespaces, timestamps)

Example:
```json
{
  "plugin": "terraform",
  "status": "high",
  "messages": ["S3 bucket prod-artifacts is public"],
  "metadata": {
    "resource_id": "arn:aws:s3:::prod-artifacts",
    "category": "exposure"
  }
}
2) Grouping
Findings are grouped by:

Resource ID

Namespace / environment

Category (drift, exposure, performance, etc.)

Grouping allows multiple plugin results about the same resource to be merged.

3) Linking
The engine looks for cross-plugin relationships:

Terraform drift → Kubernetes pod crash → CI/CD deployment failure

Helm values drift → GitOps OutOfSync → Service endpoint unreachable

Terraform IAM change → Secrets exposure → Cost spike from unexpected service usage

Linking rules are defined in correlator.py and can be extended.

4) Regression Detection
Correlator loads previous run’s results from .snapcheck/history.

Compares findings:

New — Appeared in current run, not present before.

Resolved — Present before, not in current run.

Persisting — Appears in both runs.

5) Storyline Generation
For each linked group, a human-readable storyline is generated:

csharp
Copy
Edit
Terraform drift on S3 bucket prod-artifacts → Helm upgrade failed for chart static-site → CI job 'deploy' failed (artifact missing) → AWS S3 cost increased 45% this month
Severity of storyline = max severity of linked issues.

6) Output
Storylines are added to:

Terminal output (summary table)

Markdown report

HTML dashboard “Correlation” section

Each storyline includes:

Root cause indicator

List of related findings

Regression tag (new/resolved/persisting)

Extending the Correlator
Adding a New Linking Rule
Edit core/correlator.py.

In link_findings() function, add a condition:

python
Copy
Edit
if tf.category == "drift" and helm.status == "failed_upgrade":
    storyline = f"Terraform drift on {tf.resource_id} → Helm upgrade failed for {helm.release}"
    add_storyline(storyline, severity=max(tf.status, helm.status))
Re-run tests to verify.

Adding a New Regression Type
Modify detect_regressions() to introduce new classification logic.

Example: Tag “intermittent” if finding appears sporadically over last 5 runs.

Best Practices for Developers
Keep rules deterministic — avoid random or non-repeatable correlations.

Prefer metadata matching over message string matching.

Always include severity escalation logic (max() of related severities).

Ensure new rules are covered in regression tests.

Example Storylines
Root Cause: Terraform drift on EC2 instance type → Symptoms: Kubernetes pod OOMKilled, CI/CD deploy latency.

pgsql
Copy
Edit
[HIGH] EC2 i-abc123 changed instance type (t3.large → t3.medium) without Terraform apply
       ↓
[HIGH] Pod payments-api in namespace prod restarted 12 times due to OOMKilled
       ↓
[MEDIUM] Deployment job 'deploy-payments' exceeded 40 min runtime
Root Cause: Public S3 bucket created manually → Symptoms: AWS S3 cost spike, Secrets leak.

pgsql
Copy
Edit
[CRITICAL] S3 bucket prod-logs is public (policy allows s3:GetObject to *)
       ↓
[HIGH] AWS S3 monthly spend increased 80% vs last month
       ↓
[CRITICAL] AWS Access Key found in repo src/config.py
Related Docs
Architecture

Plugin Reference

Security