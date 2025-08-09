# Helm Plugin

The Helm plugin audits **Helm releases** deployed in one or more Kubernetes namespaces.  
It identifies **failed or pending releases**, **outdated charts**, and **values drift** between the deployed release and the local chart values.

---

## Capabilities (v1.0)

- Enumerates Helm releases in target namespaces.
- Flags releases with failed/pending upgrades or installs.
- Detects outdated charts (compares deployed chart version to latest available).
- Compares deployed release values against local `values.yaml` for drift detection.
- Surfaces namespace coverage gaps (expected release missing in cluster).

---

## Prerequisites

- Access to target cluster via `kubeconfig` (same as Kubernetes plugin).
- `helm` CLI not required — SnapCheck uses the Kubernetes API to inspect Helm release secrets/configmaps.
- Optional local chart paths for drift comparison.

---

## Profile Configuration

```yaml
modules:
  - helm

helm:
  namespaces: [default, monitoring]
  checks:
    release_status: true
    chart_version: true
    values_drift: true
    missing_expected: true
  expected_releases:
    - name: monitoring-stack
      namespace: monitoring
    - name: ingress-controller
      namespace: kube-system
  local_charts:
    - name: monitoring-stack
      path: ./charts/monitoring-stack
Checks (Detailed)
1) Release Status
Signals:

Status failed, pending-upgrade, pending-install, pending-rollback.

Revision mismatch vs expected.

Severity:

Failed release: critical

Pending > 5 minutes: high

Rollback in progress: medium

2) Chart Version
Signals:

Deployed chart version is older than latest available in repo.

Latest version fetched from Helm repo index (if reachable).

Severity:

Major version behind: high

Minor/patch version behind: medium

3) Values Drift
Signals:

Differences between deployed values and local values.yaml.

Ignores non-functional changes (metadata, timestamps).

Severity:

Critical config drift (e.g., resource limits reduced, replicas changed in prod): high

Cosmetic drift (labels, annotations): low

4) Missing Expected Releases
Signals:

A release listed in expected_releases not found in cluster.

Severity:

Missing critical release (e.g., monitoring, ingress): critical

Missing non-critical: medium

Output Schema
json
Copy
Edit
{
  "plugin": "helm",
  "status": "high",
  "messages": [
    "Release monitoring-stack in namespace monitoring is using chart version 1.2.0 (latest is 1.4.1)"
  ],
  "metadata": {
    "release": "monitoring-stack",
    "namespace": "monitoring",
    "category": "chart_version",
    "deployed_version": "1.2.0",
    "latest_version": "1.4.1"
  }
}
Example Findings
Failed Release
less
Copy
Edit
[CRITICAL][helm] Release ingress-controller in kube-system failed (status: failed)
Outdated Chart
pgsql
Copy
Edit
[HIGH][helm] Release monitoring-stack in monitoring is using chart version 1.2.0 (latest: 1.4.1)
Values Drift
arduino
Copy
Edit
[HIGH][helm] Values drift detected for release api-gateway in prod: replicas=2 (expected 3)
Missing Release
arduino
Copy
Edit
[CRITICAL][helm] Expected release monitoring-stack in namespace monitoring not found
Execution Flow
Enumerate namespaces from profile.

Discover releases by reading Helm release secrets/configmaps.

Check release status and timestamps.

Compare chart versions against latest available in repo.

If local chart path provided, load values.yaml and compare with deployed values.

Check for missing expected releases.

Emit AuditResult list.

Performance & Limits
Drift comparison cost grows with size of values files.

Chart version check may be skipped if Helm repo unreachable.

Can run without internet — only skips version check.

Security Notes
Requires read-only access to Helm release secrets/configmaps in target namespaces.

Minimal RBAC example:

yaml
Copy
Edit
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: snapcheck-helm-read
  namespace: monitoring
rules:
  - apiGroups: [""]
    resources: ["secrets", "configmaps"]
    verbs: ["get", "list"]
Troubleshooting
Symptom	Cause	Resolution
"No releases found"	Namespace empty or wrong kubeconfig context	Verify context and namespace list
Version check skipped	Repo unreachable	Ensure network access to Helm repo
Drift false positives	Default values mismatch	Use same values.yaml as deployed
Missing expected release	Intentional in this env	Update expected_releases in profile

Examples
Audit Default + Monitoring Namespaces
bash
Copy
Edit
snapcheck run audit --modules helm --profile profiles/prod.yaml
Include Local Chart for Drift Detection
yaml
Copy
Edit
helm:
  namespaces: [default]
  local_charts:
    - name: api-gateway
      path: ./charts/api-gateway
Related Docs
CLI: snapcheck run

Architecture

Security

Kubernetes Plugin