# Contributing Guide

Welcome to SnapCheck! This guide explains how to propose changes, add plugins, write tests, and uphold security and quality standards.

---

## 1. Philosophy

- **Read-only by default** — no changes to user systems.
- **Deterministic** — repeatable results for the same inputs.
- **Fail isolated** — one plugin must not crash the run.
- **Secure-first** — least-privilege access, no secret persistence.
- **Docs with code** — every change ships with docs and examples.

---

## 2. Project Layout (high level)

snapcheck/
├─ core/
│ ├─ correlator.py
│ ├─ models.py # AuditResult, enums
│ └─ snapshot_diff.py
├─ plugins/
│ ├─ terraform.py
│ ├─ kubernetes.py
│ ├─ helm.py
│ ├─ ci_cd.py
│ ├─ docker.py
│ ├─ secrets.py
│ ├─ cost.py
│ └─ gitops.py
├─ output/
│ ├─ templates/
│ │ ├─ report_template.html
│ │ ├─ partials/
│ │ └─ markdown_report.py
│ └─ html_report.py
├─ utils/
│ ├─ secret_loader.py
│ ├─ state_loader.py
│ └─ publisher.py
├─ main.py # CLI (click)
├─ docs/ # Documentation (this site)
└─ .snapcheck/ # Run artifacts (generated)

markdown
Copy
Edit

---

## 3. Branching & Commits

- **Branch naming:** `feat/<area>-<short-desc>`, `fix/<area>-<issue>`, `docs/<section>`.
- **Commit style:** Conventional commits (recommended):
  - `feat(ci_cd): add flakiness threshold option`
  - `fix(kubernetes): handle headless services in endpoint check`
  - `docs(helm): clarify version check behavior`
- Reference issues with `#123` in commit body when relevant.

---

## 4. Coding Standards

- **Language:** Python 3.9+
- **Style:** `black` + `isort` (suggested), `flake8` or `ruff` for linting.
- **Typing:** Add type hints; run `mypy` where feasible.
- **Errors:** Never raise uncaught exceptions out of plugins — return `AuditResult(status=ERROR, messages=[...])`.
- **Logging:** Prefer structured logs; avoid printing secrets. Use minimal logging in plugin loops to avoid noise.

---

## 5. Plugin Contract

Every plugin must implement:

```python
def run_check(config: dict) -> Union[AuditResult, List[AuditResult]]:
    """
    - Read-only checks only
    - Must tolerate partial/missing credentials
    - Must catch errors and return status=ERROR on failure
    - Respect config.get('demo_mode') to avoid live calls
    """
Required behaviors

Idempotent for the same inputs.

Time-bounded (no infinite waits; add client timeouts).

Rate-limit aware (retry with backoff; cap requests).

Mask sensitive data before putting anything into messages/metadata.

6. Configuration & Schema
Profile keys should be namespaced per plugin:

kubernetes.namespaces, terraform.state_path, ci_cd.repo, etc.

Provide sane defaults and zero-config behavior where safe.

Validate required keys at runtime; produce a single ERROR result if invalid.

Example

python
Copy
Edit
cfg = (config or {}).get('kubernetes', {})
namespaces = cfg.get('namespaces')
if namespaces is None:
    namespaces = []  # means "all"
7. Dependencies Policy
Prefer the standard library; add third-party deps sparingly.

Any new dependency must be:

OSI-approved license, actively maintained.

Pinned in requirements.txt.

Justified in PR description (why it’s needed, alternatives considered).

8. Security Requirements
Do not log or store secrets. Use utils/secret_loader.py and mask values before output.

Use least-privilege scopes in docs and code examples.

Respect security.enabled, https_only, and static-asset gating in web features.

Share links must use expiring JWT signed with SNAPCHECK_SECRET_KEY.

9. Testing
9.1 Unit Tests
Scope: pure functions, simple adapters.

Mock external services and environment variables.

9.2 Integration Tests
Use recorded fixtures for APIs (e.g., VCR.py) or stub servers.

Validate plugin outputs (AuditResult shape and severity mapping).

9.3 Correlator Tests
Provide inputs from multiple plugins; assert storyline and severity.

Example PyTest structure

pgsql
Copy
Edit
tests/
├─ test_terraform_plugin.py
├─ test_kubernetes_plugin.py
├─ test_correlator.py
└─ fixtures/
   ├─ tfstate_small.json
   ├─ k8s_pods_list.json
   └─ github_runs.json
Sample test snippet

python
Copy
Edit
def test_public_s3_finding():
    from plugins.terraform import run_check
    cfg = {"terraform": {"state_path": "tests/fixtures/tfstate_small.json"}}
    results = run_check(cfg)
    assert any("public" in " ".join(r.messages).lower() for r in results)
10. Performance & Rate Limits
Batch API calls; use pagination.

Expose limits.* config knobs (e.g., max_runs, max_jobs, max_tags).

Implement exponential backoff on HTTP 429 and equivalent throttling signals.

For very large inventories, add sampling (drift_sample_percent).

11. Error Handling & UX
Never crash the entire run for a single plugin error.

One ERROR result per plugin is preferable to a stack trace.

Put actionable tips in messages (what to check, likely causes).

Good

css
Copy
Edit
[ERROR][gitops] Failed to list Argo CD apps (401). Tip: verify ARGOCD_TOKEN and server URL.
Avoid

sql
Copy
Edit
Traceback (most recent call last):
  File ...
12. Documentation with Code
Update or add pages under /docs/ for every feature or option:

CLI changes → /docs/cli/*.md

Plugin changes → /docs/plugins/*.md

Developer changes → /docs/developer/*.md

Include examples and minimal profiles.

Keep README.md short; link to /docs.

13. PR Checklist
 Feature/bugfix linked to issue

 Tests added/updated; all tests pass locally

 Docs updated (CLI, plugin, or developer)

 Dependencies pinned (if added)

 Secrets are masked; no plaintext in logs or code

 Backward compatible (or clearly documented breaking change)

14. Release & Versioning
Tag releases with SemVer: vMAJOR.MINOR.PATCH.

Prepare release notes:

New features

Fixes

Security / breaking changes

(Optional) Versioned docs with mike once Pages is enabled.

15. Adding a New Plugin (Step-by-Step)
Scaffold

bash
Copy
Edit
plugins/myplugin.py
docs/plugins/myplugin.md
Implement

run_check(config) with AuditResult list output.

Handle demo_mode.

Wire-up

Add to MODULE_MAP in main.py.

Add config defaults and schema notes to docs.

Test

Unit + integration tests with fixtures.

Document

Full /docs/plugins/myplugin.md with examples and RBAC/permissions.

Review

Security review (scopes, masking).

Performance review (limits, backoff).

Merge

Squash or merge commits per repo policy.

16. Code of Conduct (Short)
Be respectful and constructive.

Default to empathy. Disagree on code, not on people.

Keep discussions technical; document decisions.

17. Contact
Owners: SnapCheck Team

Security liaison: (add name/alias)

Where to file issues: GitHub Issues

Where to discuss: GitHub Discussions / Slack (if available)

Related Docs
API Reference

Correlation Engine

Plugin Reference

CLI: snapcheck run