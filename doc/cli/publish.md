# `snapcheck publish` Command

The `snapcheck publish` command exports the latest SnapCheck report artifacts for static hosting.  
It can output to a local folder (for manual upload) or to a Git branch (e.g., `gh-pages`) for GitHub Pages.

> SnapCheck publishes **static HTML/Markdown** only. Authentication/RBAC is not enforced on static hosting; use private repos or access controls where required.

---

## Syntax

```bash
snapcheck publish [OPTIONS]
Options
Option	Description	Required	Default
--profile PATH	Path to the profile YAML file.	No*	$SNAPCHECK_PROFILE
`--target PATH	SCHEME`	Publish target: local path (e.g., ./site/) or special target gh-pages.	No
--include-history	Include .snapcheck/history/ in the published bundle.	No	Disabled
--force	Overwrite target if it already exists.	No	Disabled
--dry-run	Show what would be published without writing.	No	Disabled

* Either --profile must be provided or the SNAPCHECK_PROFILE env var must be set.

What Gets Published
By default, the following files (from the latest run) are copied:

.snapcheck/report.html

.snapcheck/report.md

Optionally: .snapcheck/history/** when --include-history is set

Examples
1) Publish to a Local Folder (Manual Hosting)
bash
Copy
Edit
snapcheck publish --target ./site
# Result:
# ./site/report.html
# ./site/report.md
Upload ./site/ to any static web host (S3, Nginx, Netlify, etc.).

2) Publish to GitHub Pages (Manual, No CI)
Requires a local Git clone of the repository with push rights.

bash
Copy
Edit
# Ensure a fresh audit exists:
snapcheck run audit --profile profiles/prod.yaml --output html,markdown

# Publish artifacts to gh-pages:
snapcheck publish --target gh-pages

# Internally, SnapCheck will:
# - Create/update a temp worktree for gh-pages
# - Copy report.html and report.md to the root
# - Commit and push changes with a standard message
Make sure your repository has Pages enabled and is serving from the gh-pages branch.
Settings ➝ Pages ➝ Branch: gh-pages (root)

3) Publish With History
bash
Copy
Edit
snapcheck publish --target ./site --include-history
# ./site/report.html
# ./site/report.md
# ./site/history/<timestamp>/
Security Considerations
Static hosting has no OAuth/RBAC. If your report contains sensitive metadata, use:

Private GitHub repo + GitHub Pages (private) (Enterprise/GHES), or

Private S3 bucket with signed URLs, or

Serve via snapcheck serve (OAuth/RBAC) instead of publishing.

Always mask secrets in SnapCheck (set mask_secrets: true in the profile).

Treat published reports as exported artifacts suitable for compliance and executive reviews.

If you need access control and audit trails for viewing, prefer the web UI behind OAuth.

Manual GitHub Pages (No snapcheck publish)
If you prefer pure Git commands:

bash
Copy
Edit
# 1) Run audit
snapcheck run audit --output html,markdown

# 2) Build a local 'site' dir
mkdir -p site
cp .snapcheck/report.html site/index.html
cp .snapcheck/report.md site/report.md

# 3) Push to gh-pages
git checkout --orphan gh-pages
git --work-tree site add --all
git --work-tree site commit -m "Publish SnapCheck report"
git push origin HEAD:gh-pages
git checkout -f main
git branch -D gh-pages
Troubleshooting
Symptom	Possible Cause	Fix
report.html missing	No prior audit	Run snapcheck run audit first
Push to gh-pages fails	No permissions or branch protection	Ensure repo write access or bypass branch protection for gh-pages
Pages shows 404	Pages not enabled or wrong branch/folder	Enable Pages and select gh-pages/root
Sensitive data visible	Serving static HTML without auth	Use private repo/Pages, signed S3 URLs, or snapcheck serve
History not included	--include-history not set	Re-run with --include-history

Exit Codes
Code	Meaning
0	Published successfully
1	Invalid target or profile
2	Nothing to publish (no artifacts found)
3	Publish failed (git or filesystem error)

Related Commands
snapcheck run

snapcheck serve

snapcheck init-profile