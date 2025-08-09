# `snapcheck init-profile` Command

The `snapcheck init-profile` command creates a new **SnapCheck profile YAML** file.  
Profiles define **which modules to run**, **where credentials come from**, **RBAC rules**, and other environment-specific settings.

---

## Syntax

```bash
snapcheck init-profile [OPTIONS]
Options
Option	Description	Required	Default
--init-name NAME	Logical name for the environment (used in reports).	Yes	None
--init-output PATH	Path where the profile YAML will be saved.	Yes	None
--quickstart	Generate a minimal working profile with placeholder values.	No	Disabled
--modules LIST	Comma-separated list of modules to enable initially.	No	all
--security-template	Pre-configured RBAC/security block (open, strict).	No	strict
--overwrite	Overwrite an existing file at --init-output.	No	Disabled

Quickstart Mode
Quickstart profiles include:

All supported modules enabled

Secrets source set to environment variables

Minimal RBAC (your email as admin, all others as viewer)

Example GitHub repo and Docker image placeholders

Example:

bash
Copy
Edit
snapcheck init-profile \
  --init-name prod \
  --init-output profiles/prod.yaml \
  --quickstart
Example Quickstart Output
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
    https_only: false

mask_secrets: true
demo_mode: false
Custom Mode
If you do not use --quickstart, SnapCheck will prompt interactively:

bash
Copy
Edit
Environment name: 
AWS region: 
Secrets source (env/vault): 
Modules to enable (comma-separated, 'all' for all modules): 
GitHub repo (org/repo): 
Docker images (comma-separated name:tag pairs): 
Enable security? (y/N): 
Allowed emails/domains: 
At the end, the profile is saved to the location specified by --init-output.

Best Practices
One profile per environment (dev, stage, prod).

Version-control profiles in a secure repo.

Use security-template strict for production:

No wildcard viewer access.

Only authorized emails/domains can view.

Store secrets in Vault or CI/CD secrets manager; use env mapping in the profile.

Usage Examples
Minimal Profile for Staging
bash
Copy
Edit
snapcheck init-profile \
  --init-name staging \
  --init-output profiles/staging.yaml \
  --quickstart
Custom Profile for Production
bash
Copy
Edit
snapcheck init-profile \
  --init-name prod \
  --init-output profiles/prod.yaml \
  --modules terraform,kubernetes,helm,secrets,cost \
  --security-template strict
Exit Codes
Code	Meaning
0	Profile created successfully
1	Invalid parameters or path
2	File already exists and --overwrite not set
3	User aborted interactive prompts

Related Commands
snapcheck run

snapcheck serve

snapcheck publish