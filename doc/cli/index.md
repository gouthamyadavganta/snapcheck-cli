SnapCheck CLI Reference
The snapcheck command-line interface is the primary way to run audits, serve reports, publish them, and manage configuration profiles.
Each subcommand is self-contained and documented below.

Global Options
sql
Copy
Edit
--version                 Show SnapCheck CLI version and exit.
--list-plugins            List all available plugin modules.
--profile PATH            Path to profile YAML (default: $SNAPCHECK_PROFILE).
--modules LIST            Comma-separated modules to run or 'all'.
--output FORMAT           Output format: terminal | markdown.
--help                    Show help for command or subcommand.
Commands
1. snapcheck run
Run a full or partial audit using the specified profile.

Usage:

bash
Copy
Edit
snapcheck run audit \
  --profile profiles/prod.yaml \
  --modules terraform,kubernetes,helm \
  --output terminal
Description:

Loads a profile YAML containing all module configs, secrets sources, and security settings.

Executes selected plugins in parallel where possible.

Produces:

Terminal output (summary table + key issues)

.snapcheck/report.html (interactive HTML dashboard)

.snapcheck/report.md (Markdown report)

History entry in .snapcheck/history/

Flags:

--modules → specify subset of modules to run.

--output → choose terminal or markdown.

Examples:

bash
Copy
Edit
# Run all modules
snapcheck run audit --profile profiles/dev.yaml --modules all

# Run only Terraform and CI/CD modules
snapcheck run audit --modules terraform,ci_cd
2. snapcheck serve
Start the SnapCheck web dashboard locally.

Usage:

bash
Copy
Edit
snapcheck serve --host 0.0.0.0 --port 8000
Description:

Serves the latest generated .snapcheck/report.html via FastAPI.

OAuth + RBAC enforced if security.enabled: true in profile.

Cookies stored in signed Starlette sessions.

Flags:

--host → address to bind (default: 127.0.0.1)

--port → port to listen on (default: 8000)

--no-reload → disable auto-reload (recommended for prod).

Example:

bash
Copy
Edit
snapcheck serve --no-reload
3. snapcheck publish
Publish the latest audit report to GitHub Pages.

Usage:

bash
Copy
Edit
snapcheck publish --profile profiles/prod.yaml
Description:

Requires:

GITHUB_TOKEN env var with repo + read:org if needed.

github_repo key in profile YAML.

Pushes report.html to gh-pages branch.

Can publish to private or public Pages depending on repo settings.

Example:

bash
Copy
Edit
export GITHUB_TOKEN=ghp_xxxxx
snapcheck publish
4. snapcheck init-profile
Generate a new profile YAML with sensible defaults.

Usage:

bash
Copy
Edit
snapcheck init-profile \
  --init-name prod \
  --init-output profiles/prod.yaml \
  --quickstart
Description:

Creates a ready-to-edit profile file with:

All supported module sections.

Default security settings.

Example secrets and RBAC config.

Use --quickstart to skip prompts and output defaults.

Flags:

--init-name → profile name.

--init-output → path to save profile.

--quickstart → generate without interactive prompts.

5. snapcheck share
Create a signed, expiring share link for the last audit report.

Usage:

bash
Copy
Edit
snapcheck share --ttl 3600
Description:

Generates a JWT-based link to /shared?token=...

TTL (in seconds) controls link expiration.

View-only mode — no login required.

Flags:

--ttl → token validity in seconds (default: 3600).

Environment Variables
Variable	Purpose
SNAPCHECK_PROFILE	Default profile path if --profile not passed.
SNAPCHECK_OAUTH_CLIENT_ID	GitHub OAuth App Client ID.
SNAPCHECK_OAUTH_CLIENT_SECRET	GitHub OAuth App Client Secret.
SNAPCHECK_SECRET_KEY	Session signing key.
GITHUB_TOKEN	Used by CI/CD, Docker, Secrets plugins.
AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY	Required for Cost plugin.
ARGOCD_TOKEN	Required for GitOps plugin (if API mode).