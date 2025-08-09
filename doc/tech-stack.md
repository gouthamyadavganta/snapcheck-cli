SnapCheck Technical Stack
This document details every major technology used in SnapCheck, why we selected it, how it is integrated, and any security/operational considerations.
It serves as both an engineering reference and a security audit artifact.

1. Core Runtime & CLI Framework
Python 3.11+
What: Primary programming language and runtime.

Why we chose it:

Mature ecosystem for DevOps/MLOps tooling.

Strong library support for AWS, Kubernetes, CI/CD integrations.

Excellent community and long-term maintenance guarantees.

How we use it:

All CLI commands, plugins, and correlator logic are implemented in Python.

venv-based installs for isolation.

Requirements pinned for reproducibility.

Security/Operational Notes:

Keep runtime patched to latest security releases.

Avoid unpinned dependencies in production environments.

Click
What: Python package for building command-line interfaces.

Why we chose it:

Declarative syntax with decorators.

Native support for nested commands and options.

Clear help output for end-users.

How we use it:

Implements snapcheck run, snapcheck serve, snapcheck publish, snapcheck init-profile.

Security Notes: No direct security concerns; operates only on user input.

Rich
What: Terminal rendering library for Python.

Why: Rich formatting for audit results; supports tables, color coding, live progress bars.

How we use it: Displays audit results with severity highlights in terminal output.

Jinja2
What: Templating engine for HTML & Markdown.

Why:

Lightweight, fast, widely adopted.

Separates presentation from logic.

How we use it:

HTML reports (report_template.html, TailwindCSS, Chart.js integration).

Markdown reports for offline/portable sharing.

Security Notes: Templates are static; no untrusted user inputs are rendered.

2. Cloud & API Integrations
boto3 (AWS SDK)
What: Official AWS SDK for Python.

Why: Industry standard for AWS automation; robust service coverage.

How we use it:

AWS Cost Explorer API for cost plugin.

S3 access for Terraform state and MLflow artifacts (future).

Security Notes:

Read-only IAM policies recommended.

Credentials sourced via env/vault; no persistence.

GitHub REST API
What: GitHub’s official API for repository and actions data.

Why:

Native way to access CI/CD job history, secrets inventory, repo settings.

How we use it:

CI/CD plugin: job metrics, branch protection checks.

Secrets plugin: list and validate GitHub Actions secrets.

Security Notes:

Token scope: repo + read:actions only.

Argo CD API
What: API for querying GitOps app states.

Why:

Direct insight into app health, sync status, drift detection.

How we use it:

GitOps plugin for app health and drift metrics.

Security Notes:

Token scope: read-only; ideally tied to a service account.

3. Kubernetes & Helm
Kubernetes Python Client
What: Official Kubernetes API client for Python.

Why:

Maintained by Kubernetes SIGs; full API coverage.

How we use it:

Kubernetes plugin: node/pod health, PVC checks, DNS/service reachability.

Security Notes:

Uses kubeconfig from profile; read-only permissions.

Helm CLI (via subprocess)
What: CNCF’s package manager for Kubernetes.

Why:

Native release management; values drift detection.

How we use it:

Helm plugin: list releases, check status, detect outdated charts.

Security Notes:

No write commands executed; read-only.

4. Security & Auth
GitHub OAuth2
What: OAuth-based authentication using GitHub as the identity provider.

Why:

Low friction for engineers already using GitHub.

No password storage in SnapCheck.

How we use it:

snapcheck serve login flow.

Optional org allowlists for enterprise use.

Security Notes:

Scopes: read:user, user:email, read:org (optional).

Session cookies signed with SNAPCHECK_SECRET_KEY.

Starlette Sessions
What: Lightweight session management for FastAPI.

Why: Minimal, secure cookie-based session store.

How we use it:

Stores user role, OAuth state, and login timestamp.

Security Notes:

https_only: true in production; cookies same-site=lax.

JWT (PyJWT)
What: JSON Web Tokens for temporary share links.

Why: Self-contained, signed, expiring access tokens.

How we use it:

/share endpoint generates expiring view-only report links.

5. Web UI
FastAPI
What: Modern async Python web framework.

Why:

Fast, minimal, OpenAPI by default.

Excellent for API + static file serving.

How we use it:

Serves HTML reports with OAuth gating.

Manages /login, /auth, /logout, /share.

Security Notes: Runs behind TLS in production.

TailwindCSS
What: Utility-first CSS framework.

Why:

Easy to keep consistent UI without custom CSS overhead.

How we use it:

Styles HTML report and dashboard.

Security Notes: Static build, no runtime risk.

Chart.js
What: JavaScript charting library.

Why: Simple integration, interactive charts.

How we use it:

Visualizes cost trends, drift history, and severity counts.

6. Packaging & Documentation
pip / venv
What: Python packaging and virtual environment tools.

Why:

Isolation from system Python.

Easy cross-platform setup.

How we use it:

Install SnapCheck and dependencies in project-local venv.

Markdown & Mermaid
What: Lightweight docs format + diagrams.

Why:

Works with GitHub rendering.

Easy for engineering teams to maintain.

How we use it:

All docs in /docs folder; diagrams in README and docs pages.

7. Deployment / Publishing
GitHub Pages
What: Static site hosting from GitHub.

Why: Free, simple way to publish reports securely (private repo or OAuth-gated).

How we use it:

snapcheck publish pushes HTML report to Pages branch.

Security Notes: Keep repo private if reports contain sensitive info.