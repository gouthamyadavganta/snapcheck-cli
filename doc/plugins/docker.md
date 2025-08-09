# Docker Plugin

The Docker plugin audits **container images in remote registries** such as Docker Hub or GitHub Container Registry (GHCR).  
It identifies **tag inconsistencies, outdated images, missing metadata, and known CVEs** without requiring a local Docker daemon.

---

## Capabilities (v1.0)

- Lists available image tags in a remote registry.
- Retrieves metadata (size, digest, creation date) for each tag.
- Detects outdated images (compared to latest or date thresholds).
- Identifies missing or stale `latest` tags.
- (Optional) Performs CVE lookups using public vulnerability databases.

---

## Prerequisites

- Network access to the target registry.
- Authentication for private images:
  - **Docker Hub:** Username/password or access token.
  - **GHCR:** GitHub token with `read:packages` scope.

Example:
```bash
export DOCKER_USERNAME=youruser
export DOCKER_PASSWORD=yourpassword
# or for GHCR:
export GITHUB_TOKEN=ghp_...
Profile Configuration
yaml
Copy
Edit
modules:
  - docker

docker:
  images:
    - name: yourorg/yourimage
      tags: [latest, prod, dev]
    - name: ghcr.io/yourorg/api-service
      tags: [latest, stable]
  checks:
    tag_existence: true
    outdated_tags: true
    cve_lookup: true
  limits:
    max_tags: 50
  cve:
    provider: osv       # osv|trivy|none
    severity_threshold: high
Checks (Detailed)
1) Tag Existence
Signals:

Missing specified tag in registry.

Missing latest tag (if convention used).

Severity:

Missing latest: medium

Missing environment tag (prod, stable): high

2) Outdated Tags
Signals:

Tag older than configured age threshold (default: 180 days).

Latest tag not updated in sync with recent releases.

Severity:

Older than 180 days: medium

Older than 365 days: high

3) CVE Lookup (Optional)
Signals:

Image layers contain packages with known vulnerabilities.

Vulnerability severity above configured threshold.

Severity:

CVSS ≥ 9.0: critical

CVSS 7.0–8.9: high

CVSS 4.0–6.9: medium

Output Schema
json
Copy
Edit
{
  "plugin": "docker",
  "status": "high",
  "messages": [
    "Image yourorg/yourimage:prod last updated 400 days ago"
  ],
  "metadata": {
    "image": "yourorg/yourimage",
    "tag": "prod",
    "last_updated_days": 400
  }
}
Example Findings
Missing Tag
csharp
Copy
Edit
[HIGH][docker] Image yourorg/yourimage is missing tag 'prod'
Outdated Image
css
Copy
Edit
[HIGH][docker] Image yourorg/yourimage:prod last updated 400 days ago
Critical CVE
css
Copy
Edit
[CRITICAL][docker] Image ghcr.io/yourorg/api-service:latest contains CVE-2023-12345 (CVSS 9.8)
Execution Flow
Authenticate to registry if credentials provided.

Fetch tags for each image (respecting max_tags limit).

Retrieve metadata (digest, size, last_updated).

Run checks for missing/outdated tags.

If enabled, run CVE lookup via configured provider.

Emit AuditResult list.

Performance & Limits
CVE lookup adds time; disable for faster scans.

max_tags prevents large repos from slowing audits.

Public registry rate limits apply (Docker Hub unauthenticated: ~100 pulls/day).

Security Notes
Store credentials in environment variables or vault — never in profile YAML.

Use read-only tokens where supported.

CVE scanning pulls image metadata, not the full image.

Troubleshooting
Symptom	Cause	Resolution
Authentication error	Wrong username/token	Check registry credentials
Missing tags in results	Tag list truncated	Increase max_tags
CVE check skipped	Provider unavailable	Switch to another provider or disable CVE lookup
Rate limit exceeded	Excessive unauthenticated requests	Authenticate to registry

Examples
Audit Public Docker Hub Image
bash
Copy
Edit
snapcheck run audit --modules docker --profile profiles/prod.yaml
Audit Private GHCR Image with CVE Scan
yaml
Copy
Edit
docker:
  images:
    - name: ghcr.io/yourorg/api-service
      tags: [latest, stable]
  checks:
    cve_lookup: true
  cve:
    provider: osv
bash
Copy
Edit
export GITHUB_TOKEN=ghp_...
Related Docs
CLI: snapcheck run

Architecture

Security