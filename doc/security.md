# Security Model

SnapCheck is designed with a **security-first, read-only architecture** to protect both the systems it audits and the sensitive data it processes.  
This document explains how authentication, authorization, secret management, and data protection are handled in v1.0.

---

## 1. Security Principles

1. **Least Privilege**  
   SnapCheck requires only the minimal API scopes or permissions necessary for read-only audits.
   
2. **Read-Only by Default**  
   Plugins do not make changes to infrastructure, workloads, or pipelines.

3. **No Persistent Secrets**  
   Credentials are never written to disk by SnapCheck. They are read from environment variables or a supported secrets vault.

4. **Explicit Access Control**  
   The web UI is gated by OAuth + RBAC, and share links are time-bound.

5. **Defense in Depth**  
   Authentication, authorization, session hardening, static asset gating, and audit logging all work together to reduce exposure.

---

## 2. Authentication

### 2.1 Method
- **GitHub OAuth 2.0** is the only supported authentication method in v1.0.
- OAuth flows are initiated when a user accesses the web UI (`snapcheck serve`).
- After authentication, SnapCheck stores a signed session cookie.

### 2.2 OAuth Scopes
- **Required:**
  - `read:user` — Basic profile data
  - `user:email` — Primary verified email (used for RBAC)
- **Optional:**
  - `read:org` — Needed only if RBAC allowlists an entire organization

---

## 3. Authorization (RBAC)

### 3.1 Roles
- **viewer** — Read-only report access
- **engineer** — Can generate share links
- **admin** — Full access, including changing RBAC in profile

### 3.2 RBAC Mapping
- Configured in the profile YAML under `security.rbac`:
```yaml
security:
  rbac:
    "alice@company.com": admin
    "@company.com": engineer
    "*": viewer
Supports:

Exact email match (alice@company.com)

Domain wildcard (@company.com)

Global wildcard (*)

3.3 Deny-by-Default
If no RBAC entry matches a user, access is denied.

4. Sessions
4.1 Storage
Sessions are stored in signed cookies.

Key: security.session.cookie_name (default: snapcheck_session)

4.2 Security Settings
Signature: SNAPCHECK_SECRET_KEY (32+ random bytes, rotated via vault)

Cookie Flags:

SameSite = lax (configurable)

Secure = true in production (enforces HTTPS)

Expiry: security.session.max_age_seconds (default: 24 hours)

5. Secrets Handling
5.1 Sources
Environment Variables (default)

Vault (HashiCorp Vault integration planned)

No secrets are read from files inside the repo.

5.2 Scope Minimization
AWS keys: Read-only for Cost Explorer

GitHub token: Repo + actions read scopes only

ArgoCD token: Read-only API token

5.3 Masking
All detected secrets in reports are masked before rendering (even if not labeled “password”).

Example:
AKIAIOSFODNN7EXAMPLE → AKIA...MPLE

6. Static Asset Protection
All static HTML/CSS/JS served by snapcheck serve is auth-gated.

Prevents unauthenticated scraping of embedded report data.

Access is denied with 401/403 if not logged in.

7. Share Links
7.1 Mechanism
Expiring JWT tokens embedded in URL: /share?token=<jwt>

Tokens are signed with SNAPCHECK_SECRET_KEY.

7.2 Policy
Expiry (TTL) is configured in profile.

Share links are view-only and bypass OAuth login.

8. Audit Logging
Events Logged:

Successful logins

Failed logins

Report views

Share link creation and usage

Storage Options:

File

SQLite

S3 (future)

Disabled by default; enable in profile for compliance use.

9. Recommended Hardening
Rotate SNAPCHECK_SECRET_KEY regularly via vault.

Use https_only: true in production.

Enable audit logging for access traceability.

Restrict GitHub token scopes to read-only.

Pin Python dependencies to avoid supply chain attacks.

Related Documentation
Architecture

Getting Started

Operations Guide