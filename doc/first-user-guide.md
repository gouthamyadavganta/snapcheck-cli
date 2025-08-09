# SnapCheck — First User Guide

This guide walks a **first-time SnapCheck user** through setup, running their first audit, and exploring the results.

---

## 1️⃣ What You’ll Need

Before starting, make sure you have:

- **Python 3.9+** installed
- Access to:
  - Terraform state (local `.tfstate` or remote S3/Dynamo)
  - Kubernetes cluster (via `~/.kube/config`)
  - GitHub repo (for CI/CD scans)
  - AWS account (for cost + Terraform drift checks)
  - Argo CD (optional for GitOps checks)
- **Credentials** for your modules:
  - AWS: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
  - GitHub: `GITHUB_TOKEN` (repo read, actions read, branch protection read)
  - ArgoCD: `ARGOCD_TOKEN` (read-only apps access)
- (Optional) Docker registry credentials if scanning private images.

---

## 2️⃣ Install SnapCheck

The easiest way to install SnapCheck is via [pipx](https://pipx.pypa.io/), which installs it in an isolated environment and makes it available system-wide.

```bash
pip install --user pipx   # if you don’t already have pipx
pipx ensurepath           # make sure pipx bin directory is in PATH
pipx install snapcheck-cli
snapcheck --help
Windows users:
If snapcheck is not recognized after install, close and reopen your terminal, or run:

powershell

pipx ensurepath
Upgrading SnapCheck


pipx upgrade snapcheck-cli
Uninstalling SnapCheck


pipx uninstall snapcheck-cli
For Contributors (install from source)
If you want to work on SnapCheck itself, see the “From Source” section in the main README.md.

3️⃣ Create a Profile
Profiles define what SnapCheck audits and where it connects.

Quickstart Profile

snapcheck init-profile --init-name prod \
    --init-output profiles/prod.yaml \
    --quickstart
This will create a profiles/prod.yaml with:

Modules: terraform, kubernetes, helm, ci_cd, docker, secrets, cost, gitops

Default namespaces for Helm/K8s scans

Placeholders for your credentials

4️⃣ Add Credentials
Edit profiles/prod.yaml and update the relevant fields.
We recommend using environment variables instead of hardcoding secrets.


export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export GITHUB_TOKEN=...
export ARGOCD_TOKEN=...
export SNAPCHECK_PROFILE=profiles/prod.yaml

5️⃣ Run Your First Audit

snapcheck run audit --modules all --output terminal
Example output:


🚀 SnapCheck Audit Complete — 18 findings, 4 critical
Terraform: 2 drifted resources
Kubernetes: 1 CrashLoopBackOff pod
AWS Cost: +22% this month

6️⃣ View Reports

HTML Dashboard

snapcheck serve
Then open http://127.0.0.1:8000 in your browser.

Markdown Report

snapcheck run audit --output markdown > report.md
7️⃣ Understanding the Results
SnapCheck organizes results by plugin:

Terraform: Drift, IAM issues, stale resources

Kubernetes: Node/pod health, PVC issues, DNS/service checks

Helm: Failed/outdated releases, values drift

CI/CD: Slow/flaky jobs, branch protection gaps

Docker: Registry scan, tag list, CVE metadata

Secrets: GitHub/K8s secrets, regex leak detection

Cost: AWS monthly spend by service

GitOps: ArgoCD sync/health status

The Correlation Engine links these into root cause storylines.

8️⃣ Tips for First-Time Users
Start small: Run --modules terraform,kubernetes first to test credentials.

Use --test-mode for demo results without touching real infra.

Secure your reports: If publishing to GitHub Pages, use a private repo or enable OAuth/RBAC.

9️⃣ Next Steps
Explore docs/plugins/ for per-module deep dives.

Set up publishing to share reports automatically.

Review security.md for hardening and compliance.

✅ You’ve run your first SnapCheck audit.
From here, you can integrate it into your team workflows or scheduled monitoring.