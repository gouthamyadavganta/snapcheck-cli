# `snapcheck serve` Command

The `snapcheck serve` command starts the SnapCheck web dashboard, serving the latest audit reports over HTTP or HTTPS.  
Access is gated by **GitHub OAuth** authentication and **RBAC** authorization.

---

## Syntax

```bash
snapcheck serve [OPTIONS]
Options
Option	Description	Required	Default
--host HOST	Host/IP address to bind the server to.	No	127.0.0.1
--port PORT	Port to listen on.	No	8000
--no-reload	Disable FastAPI/UVicorn autoreload (recommended for production).	No	Disabled
--https-only	Enforce HTTPS for cookies and redirects.	No	Profile value (security.session.https_only)

Authentication
Method: GitHub OAuth 2.0

Scopes:

read:user (basic profile)

user:email (primary email for RBAC)

read:org (optional, for org allowlist)

OAuth credentials must be set as environment variables:

bash
Copy
Edit
export SNAPCHECK_OAUTH_CLIENT_ID=...
export SNAPCHECK_OAUTH_CLIENT_SECRET=...
export SNAPCHECK_SECRET_KEY="random32bytes"
Authorization (RBAC)
Defined in profile YAML under security.rbac.

Roles:

viewer — Can view reports

engineer — Can view + generate share links

admin — Full access, can update RBAC

Unmatched users are denied access.

Share Links
Accessible via /share?token=<jwt>.

Tokens are:

Signed with SNAPCHECK_SECRET_KEY

View-only

Expire per TTL set in profile

Useful for external, temporary report sharing without OAuth login.

Static Asset Gating
All static HTML/CSS/JS (including embedded data) is served only to authenticated users.

Prevents unauthenticated scraping of report contents.

Usage Examples
Serve Locally (Default)
bash
Copy
Edit
snapcheck serve
Login via: http://127.0.0.1:8000/login

Serve on All Interfaces
bash
Copy
Edit
snapcheck serve --host 0.0.0.0 --port 8080
Warning: Use HTTPS or a secure tunnel (e.g., ngrok, cloudflared) if binding to public interfaces.

Force HTTPS Cookies
bash
Copy
Edit
snapcheck serve --https-only
Recommended Production Setup
Run behind a reverse proxy (e.g., Nginx, Traefik) with TLS termination.

Keep --no-reload enabled for stability.

Set security.session.https_only: true in the profile.

Rotate SNAPCHECK_SECRET_KEY periodically via Vault.

Exit Codes
Code	Meaning
0	Server exited normally
1	Configuration or binding error
2	OAuth misconfiguration (invalid client ID/secret)

Related Commands
snapcheck run

snapcheck publish

snapcheck init-profile