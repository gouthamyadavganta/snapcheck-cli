# SnapCheck – Frequently Asked Questions (FAQ)

This page addresses common operational, configuration, and troubleshooting questions for SnapCheck.

---

## General

### **Q: What is SnapCheck?**
SnapCheck is a modular, read-only DevOps/MLOps auditing tool that inspects Terraform, Kubernetes, Helm, CI/CD pipelines (GitHub), Docker registries, Secrets, AWS Cost Explorer, and GitOps (ArgoCD).  
It correlates signals to tell **what broke, why, and when**, and outputs both HTML and Markdown reports with full history.

---

### **Q: Who is SnapCheck for?**
- **SRE / Platform Engineers** – want one-pass situational awareness across multiple systems.
- **Security / Compliance** – need portable audit artifacts for reviews and certifications.
- **Engineering Leadership** – need executive summaries, cost impact, and regression history.

---

### **Q: Is SnapCheck safe to run in production?**
Yes.  
- All modules are **read-only** where possible.
- Minimal API scopes are requested.
- No plaintext secrets are stored; all are loaded from `env` or Vault at runtime.
- Reports can be **kept private** via RBAC or private GitHub Pages.

---

## Installation & Setup

### **Q: How do I install SnapCheck?**

The recommended method is via [pipx](https://pipx.pypa.io/):

```bash
pip install --user pipx   # install pipx if you don’t have it
pipx ensurepath           # add pipx bin dir to PATH
pipx install snapcheck-cli
snapcheck version

Q: How do I create my first profile?
bash
Copy
Edit
snapcheck init-profile --init-name prod --init-output profiles/prod.yaml --quickstart
Edit the generated YAML to include your AWS, GitHub, Docker, and ArgoCD details.
See Profile Reference.

Q: Where should I store profiles?
Recommended: profiles/ folder in the repo, versioned with infra code.

Do not commit secrets. Use environment variables or Vault.

Running Audits
Q: How do I run an audit?
bash
Copy
Edit
snapcheck run audit --profile profiles/prod.yaml --modules all --output terminal
Artifacts: .snapcheck/report.html + .snapcheck/history/.

Q: How do I run only some modules?
bash
Copy
Edit
snapcheck run audit --profile profiles/prod.yaml --modules terraform,kubernetes,secrets
Q: Can I test SnapCheck without real credentials?
Yes – set demo_mode: true in your profile. All modules will return synthetic data.

Web Dashboard
Q: How do I start the SnapCheck web dashboard?
bash
Copy
Edit
snapcheck serve
Login at http://127.0.0.1:8000/login (requires GitHub OAuth client ID/secret in env).

Q: How do I secure the dashboard?
Enable security.enabled: true in your profile.

Configure allowed_emails and rbac.

Set https_only: true in security.session when behind TLS.

Q: Can I share a report without giving dashboard access?
Yes:

bash
Copy
Edit
snapcheck share --profile profiles/prod.yaml --ttl 3600
Generates a signed, expiring view-only URL.

Credentials & Security
Q: How does SnapCheck get credentials?
Environment variables (secrets_source: env)

Vault (secrets_source: vault, vault_addr, vault_token_env)

Q: What API scopes are required?
GitHub: repo (read), read:org if org allowlist used, read:actions

AWS: Cost Explorer ce:Get*, S3 GetObject if remote tfstate

ArgoCD: applications read scope

Q: Does SnapCheck store my secrets?
No. Secrets are only in-memory during the run. They are never written to disk or embedded in reports.

Troubleshooting
Q: Terraform module says “state not found”
Check:

terraform_state in profile points to a valid local path or S3 URL.

If S3: ensure AWS_ACCESS_KEY_ID/AWS_SECRET_ACCESS_KEY are set and have read permission.

Q: Kubernetes module says “cannot load kubeconfig”
Verify kubeconfig path in profile is correct.

Check that your local kubectl can connect to the cluster.

Q: CI/CD module says “API rate limit exceeded”
GitHub API rate limits are per token.

Use a PAT with sufficient quota.

SnapCheck caches results per run to reduce calls.

Q: Cost module returns $0
Ensure AWS Cost Explorer is enabled in the account.

Allow 24 hours after enabling for data to populate.

Best Practices
Keep profiles per environment (dev, stage, prod).

Always enable mask_secrets: true in production.

Review allowed_emails and rbac regularly.

Store reports in private repos or behind auth.

Related
Profile Reference

Plugins Overview

Security Model

CLI Commands

pgsql
Copy
Edit
