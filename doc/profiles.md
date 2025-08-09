# SnapCheck Profile Schema Reference

A **profile** defines how SnapCheck connects to your infrastructure, what modules run,  
and how access control is enforced.  
Profiles are stored as YAML and passed to SnapCheck via the `--profile` flag or the `SNAPCHECK_PROFILE` environment variable.

---

## Example Minimal Profile

```yaml
name: prod
aws_region: us-east-1

secrets_source: env
env_secrets: [GITHUB_TOKEN, SNAPCHECK_SECRET_KEY]

modules: [terraform, kubernetes, helm, ci_cd, docker, secrets, cost, gitops]

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
    https_only: false

mask_secrets: true
demo_mode: false
Field-by-Field Reference
Key	Type	Required	Default	Description
name	string	✔	—	Human-readable profile name (appears in reports).
aws_region	string	✖	us-east-1	AWS region for API calls (Cost Explorer, etc.).
secrets_source	string (env | vault)	✔	env	How SnapCheck fetches credentials.
vault_addr	string	✖	—	Vault server address (if secrets_source: vault).
vault_token_env	string	✖	—	Env var containing Vault token.
env_secrets	list[string]	✖	—	Environment variable names to load as secrets.
modules	list[string]	✔	—	Modules to run (see Plugins). Use all for all available.
terraform_state	string	✖	—	Path/URL to .tfstate for Terraform plugin.
kubeconfig	string	✖	—	Path to kubeconfig for Kubernetes plugin.
helm_namespaces	list[string]	✖	—	Namespaces to scan for Helm releases.
ci_platform	string (github)	✖	—	CI/CD provider. Currently only github is supported.
github_repo	string	✖	—	GitHub repo in owner/name format for CI/CD checks.
docker_images	list[map]	✖	—	Images to scan. Each entry: {name, tags}.
gitops	map	✖	—	GitOps settings (see below).
mask_secrets	bool	✖	true	Whether to mask all secrets/passwords in outputs.
demo_mode	bool	✖	false	If true, uses synthetic data for all modules.
security	map	✖	see below	Access control settings (OAuth, RBAC).

Security Block (security)
Key	Type	Required	Default	Description
enabled	bool	✖	false	Enable dashboard auth gating.
auth_provider	string	✖	github	Currently only GitHub OAuth supported.
allowed_emails	list[string]	✖	—	Explicitly allowed emails.
allowed_orgs	list[string]	✖	—	Allowed GitHub orgs (requires read:org scope).
rbac	map	✖	—	Role mapping: email/domain wildcard → viewer / engineer / admin.
watermark	string	✖	—	Text watermark shown in dashboard.
session	map	✖	see below	Session cookie config.

Session Block (security.session)
Key	Type	Required	Default	Description
cookie_name	string	✖	snapcheck_session	Cookie name for auth session.
max_age_seconds	int	✖	86400	Lifetime of session in seconds.
same_site	string	✖	lax	Cookie SameSite setting (lax, strict, none).
https_only	bool	✖	true	If true, cookies only sent over HTTPS.

GitOps Block (gitops)
Key	Type	Required	Default	Description
method	string (api)	✖	—	How to connect to GitOps controller (currently ArgoCD API).
argo_server	string	✖	—	ArgoCD API server URL.
argo_token_env	string	✖	—	Env var containing ArgoCD API token.
test_mode	bool	✖	false	If true, returns synthetic GitOps data.

Module-Specific Keys
Each module may have its own extra keys.
See /docs/plugins/<module>.md for per-module config reference.

Examples:

Terraform Module
yaml
Copy
Edit
terraform_state: s3://mybucket/path/to/terraform.tfstate
Cost Module
yaml
Copy
Edit
cost:
  aws_region: us-east-1
  granularity: MONTHLY
  lookback_months: 3
  anomaly_threshold_percent: 25
Best Practices
Version-control all profiles (except secrets) alongside infrastructure code.

Separate profiles per environment (dev, staging, prod).

Use environment variables or Vault for credentials, never commit secrets.

Keep mask_secrets: true in production.

Review allowed_emails and rbac regularly.

Related
Plugins Overview

Security Model

CLI Usage