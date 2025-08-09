# SnapCheck Architecture

SnapCheck is a **modular, read-only auditing and correlation platform**.  
It is designed to run locally or in CI/CD, connect to multiple DevOps/MLOps systems, gather health and configuration data, and produce secure, shareable reports.

---

## High-Level Architecture Diagram

```mermaid
flowchart TD
    A[Profile YAML] --> B[CLI Core]
    B --> C[Plugin Executors]
    C -->|Terraform, K8s, Helm, CI/CD, Docker, Secrets, Cost, GitOps| D[Plugin Outputs]
    D --> E[Correlation Engine]
    E --> F[Report Renderer]
    F -->|HTML, Markdown, Terminal| G[Artifacts + History]
    G --> H[Web Server (FastAPI)]
    H --> I[OAuth + RBAC Layer]
    I --> J[Browser UI]
Core Components
1. CLI Core
Entry point for snapcheck run, serve, publish, and init-profile.

Loads and validates the selected profile YAML.

Injects credentials from environment variables or Vault.

Dispatches plugin execution based on profile modules.

2. Profiles
Config-as-code YAML defining:

Enabled modules

Secrets source and required keys

Target systems (repos, clusters, registries)

RBAC rules

Output/security settings

Profiles are version-controlled for reproducibility across environments (dev/stage/prod).

3. Plugins
Self-contained modules that connect to specific systems.

Always run in read-only mode.

Supported plugins in v1.0:

terraform

kubernetes

helm

ci_cd (GitHub)

docker

secrets

cost

gitops (Argo CD)

Each plugin outputs:

Structured AuditResult objects with messages, severity, and metadata.

Partial failures are isolated — one failing plugin does not halt others.

4. Correlation Engine
Consumes all plugin outputs.

Builds storylines linking cause and effect across systems.

Detects regressions compared to previous runs.

Assigns severity to each issue (critical/high/medium/low).

5. Report Renderer
Converts raw results + correlation data into:

HTML dashboard (Tailwind + Chart.js)

Markdown for text-based sharing

Terminal output for quick audits

Embeds run metadata:

Timestamp

Profile name

Module set

6. History & Diffs
Every run produces:

.snapcheck/report.html (latest)

.snapcheck/report.md (latest)

.snapcheck/history/{timestamp}/ (historical runs)

Historical data enables:

Diffing between runs

Trend analysis

Regression detection

7. Web Server
Invoked via snapcheck serve.

FastAPI app serving:

Latest report

OAuth-based login

RBAC enforcement

Optional expiring share links (/share?token=...)

8. Security Layers
Authentication: GitHub OAuth (read:user, user:email, optional read:org).

Authorization: RBAC mapping (email/domain → role).

Sessions: Signed cookies (SNAPCHECK_SECRET_KEY), TLS recommended in production.

Secrets Handling: Environment/vault only, never stored in plaintext.

Static Asset Gating: HTML/CSS/JS served only to authenticated users.

Data Flow
Profile Load: CLI reads profile and sets execution context.

Secrets Injection: Environment variables or vault populate required keys.

Plugin Execution: Each plugin connects to its target system and returns results.

Correlation: Results are merged, linked, and scored.

Report Generation: Outputs are rendered in multiple formats.

History Save: Results are archived with timestamp.

Optional Serve: Reports can be viewed via authenticated web UI.

Deployment Modes
Local Developer Workstation

Run audits directly from CLI.

View HTML in browser locally.

CI/CD Pipeline

Execute snapcheck run as a job.

Publish artifacts to GitHub Pages or internal storage.

Read-Only Ops Node

Host snapcheck serve on a secure VM for on-demand viewing.

Related Documentation
Getting Started

Security

Plugin Reference