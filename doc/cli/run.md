# `snapcheck run` Command

The `snapcheck run` command executes an audit based on the specified profile, runs the configured plugins, correlates results, and generates reports in one or more output formats.

---

## Syntax

```bash
snapcheck run <subcommand> [OPTIONS]
Subcommands:

audit — Run a full audit and generate reports.

Options
Option	Description	Required	Default
--profile PATH	Path to the profile YAML file.	Yes	None
--modules LIST	Comma-separated list of modules to run, or all.	No	all
--output FORMAT	Output format: terminal, markdown, html. Multiple formats may be specified (comma-separated).	No	terminal
--list-plugins	List all available plugins without running them.	No	N/A
--version	Show SnapCheck version and exit.	No	N/A

Profile Loading
SnapCheck requires a profile YAML to determine:

Enabled modules

Secrets source

Target infrastructure and repos

Security/RBAC settings

Example:

bash
Copy
Edit
snapcheck run audit --profile profiles/prod.yaml
Output Formats
terminal — Human-readable audit summary printed to stdout.

markdown — Structured .md file for documentation and reviews.

html — Interactive dashboard viewable in a browser.

Multiple formats example:

bash
Copy
Edit
snapcheck run audit --profile profiles/prod.yaml --output terminal,html
Module Selection
By default, SnapCheck runs all modules in the profile.

To run only specific modules:

bash
Copy
Edit
snapcheck run audit --profile profiles/prod.yaml --modules terraform,kubernetes,secrets
To list available modules without running:

bash
Copy
Edit
snapcheck run --list-plugins
Environment Variables
Required environment variables depend on enabled modules.
Common examples:

bash
Copy
Edit
export SNAPCHECK_PROFILE=profiles/prod.yaml
export SNAPCHECK_SECRET_KEY="random32bytes"
export GITHUB_TOKEN="ghp_..."   # repo + actions read
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
Examples
Full Audit
bash
Copy
Edit
snapcheck run audit --profile profiles/prod.yaml
Terraform + Secrets Only
bash
Copy
Edit
snapcheck run audit --profile profiles/dev.yaml --modules terraform,secrets
Generate HTML + Markdown
bash
Copy
Edit
snapcheck run audit --profile profiles/prod.yaml --output html,markdown
Exit Codes
Code	Meaning
0	Completed successfully (no critical SnapCheck errors)
1	Fatal error in core execution (e.g., invalid profile)
2	Plugins executed but some failed
3	Plugins executed but critical findings present

Notes
snapcheck run is read-only — it never modifies target systems.

For local testing, set demo_mode: true in the profile to generate stubbed results.

Related Commands
snapcheck serve

snapcheck publish

snapcheck init-profile