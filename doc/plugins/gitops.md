# GitOps Plugin (Argo CD)

The GitOps plugin integrates with **Argo CD** to audit the **health, sync status, and drift** of GitOps-managed applications.  
It detects **failed syncs, revision drift, missing auto-sync**, and other issues that can impact continuous delivery.

---

## Capabilities (v1.0)

- Lists all Argo CD applications in the target instance.
- Reports:
  - Sync status (`Synced`, `OutOfSync`)
  - Health status (`Healthy`, `Degraded`, `Suspended`)
  - Last sync time and revision
  - Revision drift between Git and live cluster
  - Failed sync attempts
  - Auto-sync and self-heal flags
- (Optional) Detects Helm values drift for Helm-based Argo CD apps.

---

## Prerequisites

- **Argo CD API endpoint** and token with read-only permissions:
  - Generate in Argo CD UI under *Settings → Accounts* or via CLI:
    ```bash
    argocd account generate-token --account read-only
    ```
- API server accessible from the SnapCheck environment.
- TLS certificates trusted or `insecure: true` set in profile (not recommended in prod).

---

## Profile Configuration

```yaml
modules:
  - gitops

gitops:
  platform: argocd
  server: https://argo.example.com
  token: ${ARGOCD_TOKEN}
  checks:
    sync_status: true
    health_status: true
    revision_drift: true
    failed_syncs: true
    auto_sync: true
    helm_values_drift: false
  limits:
    max_apps: 200
  insecure: false
Checks (Detailed)
1) Sync Status
Signals:

OutOfSync applications.

Severity:

Prod app out of sync: high

Non-prod out of sync: medium

2) Health Status
Signals:

Degraded or Suspended apps.

Severity:

Degraded prod app: critical

Degraded non-prod app: high

3) Revision Drift
Signals:

Git revision in Argo CD does not match target revision in source repo.

Live cluster resources differ from declared manifests.

Severity:

Drift on prod app: critical

Drift on non-prod: high

4) Failed Syncs
Signals:

Sync attempts failed in last N hours (configurable).

Severity:

Multiple failed syncs: high

5) Auto-Sync & Self-Heal
Signals:

Auto-sync disabled on critical apps.

Self-heal disabled on prod apps.

Severity:

Auto-sync off in prod: medium

Self-heal off in prod: medium

6) Helm Values Drift (Optional)
Signals:

Values in Argo CD-managed Helm chart differ from local values.yaml.

Severity:

Critical drift in prod values: high

Non-critical drift: low/medium

Output Schema
json
Copy
Edit
{
  "plugin": "gitops",
  "status": "critical",
  "messages": [
    "Application payments-api in prod is Degraded and OutOfSync"
  ],
  "metadata": {
    "app": "payments-api",
    "namespace": "argocd",
    "health": "Degraded",
    "sync": "OutOfSync",
    "revision": "abc123",
    "target_revision": "def456"
  }
}
Example Findings
Degraded and OutOfSync
csharp
Copy
Edit
[CRITICAL][gitops] Application payments-api in prod is Degraded and OutOfSync
Auto-Sync Disabled
scss
Copy
Edit
[MEDIUM][gitops] Application ingress-controller auto-sync is disabled in prod
Revision Drift
css
Copy
Edit
[HIGH][gitops] Application analytics-service target revision def456 differs from live revision abc123
Execution Flow
Authenticate to Argo CD API using token.

Fetch applications (respecting max_apps limit).

For each application:

Check sync status, health status.

Check last sync time and revision.

Optionally compare Helm values.

Emit AuditResult list.

Performance & Limits
API paging handled automatically.

Drift detection may require additional API calls per app.

Helm values drift check can be expensive; disable unless required.

Security Notes
Token should be read-only with applications, projects list/get permissions.

Do not store token directly in profile — use environment variables or vault.

Avoid setting insecure: true in production unless TLS issues cannot be resolved.

Troubleshooting
Symptom	Cause	Resolution
401 Unauthorized	Invalid or expired token	Regenerate token in Argo CD
Empty app list	Token lacks app list permission	Assign correct RBAC policy in Argo CD
Drift check slow	Large number of apps	Increase max_apps or disable drift checks

Examples
Audit All Apps
bash
Copy
Edit
snapcheck run audit --modules gitops --profile profiles/prod.yaml
Audit With Helm Values Drift
yaml
Copy
Edit
gitops:
  helm_values_drift: true
Related Docs
CLI: snapcheck run

Architecture

Security