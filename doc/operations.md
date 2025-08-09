# Operations Guide

This guide covers how to operate SnapCheck in day-to-day workflows, handle failures, manage performance, and integrate with enterprise processes.

---

## 1. Operating Modes

### 1.1 Local CLI Mode
- Run directly on a developer or engineer’s workstation.
- Best for:
  - Ad-hoc audits
  - Testing profile changes
  - Pre-deployment checks

```bash
snapcheck run audit --modules all --output terminal
1.2 Web UI Mode
Start snapcheck serve to host reports behind OAuth + RBAC.

Best for:

Shared review sessions

On-demand viewing by non-CLI users

bash
Copy
Edit
snapcheck serve
1.3 Published Reports
Use snapcheck publish to push HTML reports to a GitHub Pages branch or other static hosting.

Best for:

Asynchronous sharing

Embedding in compliance portals

bash
Copy
Edit
snapcheck publish
2. Run Frequency
On-demand: Run manually before critical changes.

Scheduled: Run daily/weekly in CI/CD pipelines.

Event-driven: Trigger after Terraform plan/apply, Helm upgrade, or pipeline failures.

3. Handling Failures
3.1 Plugin Isolation
Each plugin runs independently.

A failure in one plugin does not stop the others.

Failures are shown in the report as “Plugin Error” with details.

3.2 Common Failure Causes
Plugin	Common Cause	Resolution
Terraform	Missing or inaccessible .tfstate	Verify path or S3 backend credentials
Kubernetes	Invalid kubeconfig	Update kubeconfig context
Helm	Missing Tiller permissions	Ensure helm list access in cluster
CI/CD	Expired GitHub token	Refresh token in environment
Docker	Registry auth failure	Check credentials or public access
Secrets	Insufficient permissions for secret list	Adjust role or scope
Cost	AWS CE disabled or no cost data	Enable CE in AWS console
GitOps	Expired ArgoCD token	Generate new read-only token

4. Performance Considerations
4.1 Rate Limits
GitHub API: Rate limit based on token scope; SnapCheck backs off and retries.

AWS Cost Explorer: 1 request/sec default limit; SnapCheck batches requests.

4.2 Parallel Execution
Plugins run in parallel “where safe.”

You can limit concurrency via profile config (future feature planned).

5. History & Regression Tracking
Reports are saved to .snapcheck/history/{timestamp}/ with full raw data.

The correlator compares the latest run with the previous to detect regressions.

To compare runs manually:

bash
Copy
Edit
snapcheck run diff --from 2025-08-01T10:00 --to 2025-08-08T10:00
(Diff command planned; currently use manual file compare.)

6. Disaster Recovery
SnapCheck is stateless except for .snapcheck/ artifacts.

To restore history after a machine loss:

Backup .snapcheck/history/ and .snapcheck/report.html

Restore to same path on a new machine

Serve locally or re-publish

7. Best Practices for Operations
Keep Profiles in Git: Version-control all environment configs.

Use Short-Lived Tokens: Rotate GitHub/AWS/Argo tokens regularly.

Limit Module Scope: Only enable modules relevant to the target environment.

Run in Staging First: Validate profile changes before production runs.

Archive Reports: Store HTML/Markdown in S3 or artifact storage for compliance.

8. Example Operational Playbook
Situation	Action
Pre-deployment review	Run snapcheck run audit on target env; verify no criticals.
Weekly compliance scan	Schedule SnapCheck in CI/CD; store artifacts in S3.
Incident investigation	Run SnapCheck; use correlation storylines to trace root cause.
Cost spike detected	Run SnapCheck with cost module enabled; check cost + infra drift.
Suspected secret leak	Enable secrets module; review masked results for potential exposure.

Related Documentation
Getting Started

Architecture

Security

CLI Reference