# SnapCheck API Reference

This document describes the **internal Python API** of SnapCheck.  
It’s intended for developers extending plugins, building new integrations, or modifying the CLI.

---

## Core Data Structures

### `AuditResult`
Defined in `core/models.py`

Represents the output of a single plugin check.

```python
class AuditResult:
    plugin: str            # Name of plugin (terraform, kubernetes, etc.)
    status: StatusLevel    # Enum: CRITICAL, HIGH, MEDIUM, LOW, OK, ERROR
    messages: List[str]    # Human-readable findings
    metadata: dict         # Structured details (resource IDs, namespaces, timestamps)
    timestamp: datetime    # When finding was detected
Usage Example:

python
Copy
Edit
from core.models import AuditResult, StatusLevel

result = AuditResult(
    plugin="terraform",
    status=StatusLevel.HIGH,
    messages=["S3 bucket prod-artifacts is public"],
    metadata={"resource_id": "arn:aws:s3:::prod-artifacts", "category": "exposure"}
)
StatusLevel Enum
Represents severity.

python
Copy
Edit
class StatusLevel(Enum):
    CRITICAL = "critical"
    HIGH     = "high"
    MEDIUM   = "medium"
    LOW      = "low"
    OK       = "ok"
    ERROR    = "error"
Plugin Interface
Each plugin implements a run_check(config) function.

Signature
python
Copy
Edit
def run_check(config: dict) -> Union[AuditResult, List[AuditResult]]:
    ...
Arguments:

config — dict loaded from profile YAML (includes secrets, endpoints, modules list, etc.)

Returns:

A single AuditResult or a list of them.

Requirements:

Must not raise uncaught exceptions — catch and return AuditResult with status=ERROR.

Must support test_mode in config to avoid hitting live systems.

CLI Commands
Defined in main.py using click.

snapcheck run
python
Copy
Edit
@click.command()
@click.option('--profile', help='Path to profile YAML')
@click.option('--modules', default='all', help='Comma-separated list of modules')
@click.option('--output', type=click.Choice(['terminal','markdown','html']))
def run(profile, modules, output):
    ...
Loads config → runs plugins → passes results to correlator → renders outputs.

snapcheck serve
python
Copy
Edit
@click.command()
@click.option('--host', default='127.0.0.1')
@click.option('--port', default=8000)
@click.option('--no-reload', is_flag=True)
def serve(host, port, no_reload):
    ...
Starts FastAPI web server with OAuth + RBAC. Serves latest HTML report.

snapcheck publish
python
Copy
Edit
@click.command()
@click.option('--profile', help='Path to profile YAML')
def publish(profile):
    ...
Publishes latest .snapcheck/report.html to GitHub Pages.

Utility Modules
utils/secret_loader.py
Fetches secrets from:

Environment variables

HashiCorp Vault (if enabled)

Masks all detected secret values before output.

utils/publisher.py
Handles pushing reports to GitHub Pages (branch gh-pages).

Uses GitHub token from env.

utils/state_loader.py
Loads Terraform state from:

Local file path

S3/HTTP backend

Returns parsed JSON for Terraform plugin.

Correlation Engine API
link_findings(results: Dict[str, List[AuditResult]]) -> List[Storyline]
Groups and links findings across plugins.

Returns a list of Storyline objects with root cause vs symptom tagging.

detect_regressions(current, previous) -> RegressionReport
Compares two sets of plugin outputs and marks findings as new, resolved, or persisting.

Report Generation API
generate_html_report(plugin_results, summary, charts, gpt_summary, diff, timestamp, out_path)
Renders HTML from Jinja templates with TailwindCSS + Chart.js.

generate_markdown_report(plugin_results, summary, timestamp, out_path)
Renders Markdown for portability.

Extending the API
Adding a New Plugin
Create plugins/myplugin.py

Implement run_check(config) returning AuditResult(s)

Add "myplugin" to MODULE_MAP in main.py

Update /docs/plugins/myplugin.md with usage instructions.

Adding New Output Formats
Create a renderer in output/templates/ and call from CLI.

Related Docs
Architecture

Correlation Engine

Plugin Reference