# Secrets Plugin

The Secrets plugin audits **secrets management practices** across GitHub Actions and Kubernetes.  
It inventories secrets, detects **hardcoded credentials**, checks **secret age**, and flags **unused or risky secrets**.

---

## Capabilities (v1.0)

- **GitHub Actions secrets inventory**:
  - Lists all secrets configured in repo/org.
  - Flags secrets unused in workflows.
  - Checks last updated date (age).
- **Kubernetes secrets inventory**:
  - Lists all secrets in target namespaces.
  - Detects base64-decoded sensitive data.
  - Flags unused secrets (not mounted in any pod).
- **Regex-based leak detection** in source code:
  - AWS keys, private keys, tokens, passwords.
- **Severity tagging** for each finding.

---

## Prerequisites

- **For GitHub secrets**:
  - Token with `repo` and `admin:repo_hook` (read-only) or `secrets:read` (fine-grained).
  - For org-level secrets: token with `admin:org` (read-only).

- **For Kubernetes secrets**:
  - Access via `kubeconfig`.
  - RBAC:
    ```yaml
    - apiGroups: [""]
      resources: ["secrets", "pods"]
      verbs: ["get", "list"]
    ```

- **For code scanning**:
  - Local source directory path accessible to SnapCheck.

---

## Profile Configuration

```yaml
modules:
  - secrets

secrets:
  checks:
    github_inventory: true
    github_unused: true
    github_age: true
    k8s_inventory: true
    k8s_unused: true
    regex_leak: true
  regex_patterns:
    aws_access_key: "AKIA[0-9A-Z]{16}"
    private_key: "-----BEGIN PRIVATE KEY-----"
    generic_token: "[A-Za-z0-9_\\-]{20,}"
  github:
    repo: yourorg/yourrepo
    org: yourorg    # optional, for org secrets
  kubernetes:
    namespaces: [default, monitoring]
  age_threshold_days: 180
Checks (Detailed)
1) GitHub Secrets Inventory
Signals:

Secret exists in repo/org settings.

Severity: Informational unless unused/aged.

2) Unused GitHub Secrets
Signals:

Secret not referenced in any workflow YAML in repo.

Severity:

Unused > 180 days: medium

Unused but recently created: low

3) GitHub Secret Age
Signals:

Secret older than age_threshold_days.

Severity:

Age > 365 days: high

Age > threshold but < 365: medium

4) Kubernetes Secrets Inventory
Signals:

Secret exists in namespace.

Severity: Informational unless unused/contains sensitive key types.

5) Unused Kubernetes Secrets
Signals:

Secret not mounted or referenced by any pod in the namespace.

Severity:

Unused > 180 days: medium

Unused critical secret (e.g., db-password): high

6) Regex Leak Detection
Signals:

Pattern match in code/workflows indicating potential secret leak.

Severity:

AWS key/private key in repo: critical

Token/password in repo: high

Output Schema
json
Copy
Edit
{
  "plugin": "secrets",
  "status": "high",
  "messages": [
    "GitHub secret AWS_ACCESS_KEY_ID in repo yourorg/yourrepo is older than 365 days"
  ],
  "metadata": {
    "source": "github",
    "secret_name": "AWS_ACCESS_KEY_ID",
    "age_days": 370,
    "usage_count": 0
  }
}
Example Findings
Old GitHub Secret
csharp
Copy
Edit
[HIGH][secrets] GitHub secret AWS_ACCESS_KEY_ID in repo yourorg/yourrepo is older than 365 days
Unused Kubernetes Secret
csharp
Copy
Edit
[MEDIUM][secrets] Kubernetes secret db-password in namespace staging is unused
Leaked AWS Key in Source
css
Copy
Edit
[CRITICAL][secrets] AWS Access Key found in file src/config.py line 12
Execution Flow
GitHub Secrets:

Fetch secrets list for repo/org.

Compare against workflow usage.

Check last updated date.

Kubernetes Secrets:

Fetch secrets in namespaces.

Check if mounted in any pods.

Decode base64 for leak detection (optional).

Regex Leak Scan:

Scan repo files for configured patterns.

Ignore .gitignore patterns if specified.

Emit AuditResult list.

Performance & Limits
GitHub API limits apply (~5000 req/hr).

Large repos: regex scan may take longer; can scope to certain directories.

Kubernetes checks limited by RBAC permissions.

Security Notes
Never store secrets in profile YAML.

GitHub token should be read-only for secrets listing.

Regex matches are masked in reports to prevent exposing full secret values.

Troubleshooting
Symptom	Cause	Resolution
403 Forbidden on GitHub API	Token lacks secrets:read	Use correct scope or fine-grained token
Kubernetes secrets missing in results	RBAC denies list/get	Grant read access in target namespaces
Too many false positives in regex scan	Overly broad pattern	Refine regex_patterns in profile

Examples
Audit GitHub and K8s Secrets
bash
Copy
Edit
snapcheck run audit --modules secrets --profile profiles/prod.yaml
Regex Leak Scan Only
yaml
Copy
Edit
secrets:
  checks:
    github_inventory: false
    k8s_inventory: false
    regex_leak: true
  regex_patterns:
    private_key: "-----BEGIN PRIVATE KEY-----"
Related Docs
CLI: snapcheck run

Architecture

Security

pgsql
Copy
Edit
