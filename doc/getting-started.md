# Getting Started with SnapCheck

This guide walks you through installing SnapCheck, creating your first profile, running an audit, and viewing results.

---

## Prerequisites

Before starting, ensure you have:

- **Python 3.9+** installed (`python --version`)
- **pip** package manager
- (Recommended) **virtual environment** (`venv`)
- Access credentials for the services you plan to audit:
  - **GitHub** (repo + actions read scopes)
  - **AWS** (Cost Explorer read-only, if cost plugin is enabled)
  - **Kubernetes** (kubeconfig for cluster access)
  - **Argo CD** (read-only token, if GitOps plugin is enabled)

---

## 1. Install SnapCheck

The easiest way to install SnapCheck is via [pipx](https://pipx.pypa.io/), which ensures it’s installed in an isolated environment and available system-wide.

```bash
pip install --user pipx   # install pipx if you don’t have it
pipx ensurepath           # add pipx bin dir to PATH
pipx install snapcheck-cli
snapcheck version
Upgrading

bash
Copy
Edit
pipx upgrade snapcheck-cli
Uninstalling

bash
Copy
Edit
pipx uninstall snapcheck-cli
Windows note: If snapcheck is not recognized after install, close and reopen your terminal or run:

powershell
Copy
Edit
pipx ensurepath

2. Initialize a Profile
Profiles are YAML files that define:

Which modules/plugins run

Where credentials come from

Security and RBAC settings

Environment-specific configuration

bash
Copy
Edit
snapcheck init-profile \
  --init-name prod \
  --init-output profiles/prod.yaml \
  --quickstart
Note: The --quickstart flag generates a minimal working profile.
You can later edit profiles/prod.yaml to customize modules and RBAC.

Example Minimal Profile
yaml
Copy
Edit
name: prod
aws_region: us-east-1

secrets_source: env
env_secrets: [GITHUB_TOKEN, SNAPCHECK_SECRET_KEY]

modules:
  - terraform
  - kubernetes
  - helm
  - ci_cd
  - docker
  - secrets
  - cost
  - gitops

ci_platform: github
github_repo: yourorg/yourrepo
docker_images:
  - name: yourorg/yourimage
    tags: [latest]

security:
  enabled: true
  allowed_emails: ["you@company.com"]
  rbac:
    "you@company.com": admin
    "*": viewer
  session:
    cookie_name: snapcheck_session
    max_age_seconds: 86400
    same_site: lax
    https_only: false   # Set to true in production

mask_secrets: true
demo_mode: false
3. Set Environment Variables
Export credentials and profile location in your current shell:

bash
Copy
Edit
export SNAPCHECK_PROFILE=profiles/prod.yaml
export SNAPCHECK_OAUTH_CLIENT_ID=your_github_oauth_client_id
export SNAPCHECK_OAUTH_CLIENT_SECRET=your_github_oauth_client_secret
export SNAPCHECK_SECRET_KEY="a_very_long_random_string"
export GITHUB_TOKEN="ghp_..."   # repo + actions read

# Optional for AWS Cost Explorer
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...

# Optional for Argo CD GitOps plugin
export ARGOCD_TOKEN=...
Tip: Store these in a secure secrets manager (e.g., Vault, AWS Secrets Manager) and load them into your shell before running SnapCheck.

4. Run an Audit
bash
Copy
Edit
snapcheck run audit \
  --profile profiles/prod.yaml \
  --modules all \
  --output terminal
Artifacts generated:

.snapcheck/report.html — Full HTML dashboard

.snapcheck/report.md — Markdown report

.snapcheck/history/ — Timestamped run history for diffs and regressions

5. Serve the Dashboard Locally
bash
Copy
Edit
snapcheck serve --no-reload
Visit: http://127.0.0.1:8000/login

Authenticate with GitHub OAuth (RBAC applied automatically)

View the most recent report in your browser

