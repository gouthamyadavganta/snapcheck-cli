# CI/CD Plugin (GitHub)

The CI/CD plugin audits **pipeline health, velocity, and governance** for GitHub Actions workflows.  
It identifies **slow or failing jobs, flaky workflows, latency bottlenecks, and security policy gaps**.

---

## Capabilities (v1.0)

- Lists recent workflow runs and jobs for the target GitHub repository.
- Identifies:
  - Longest-running jobs and averages.
  - Job flakiness (intermittent failures).
  - Commit-to-deploy latency.
  - Branch protection rule compliance.
  - Bot-triggered jobs (e.g., Dependabot).
  - Artifact size trends.
  - Secrets usage within workflows.

---

## Prerequisites

- GitHub repository in the form: `owner/repo`.
- GitHub personal access token or fine-grained token with:
  - `repo` (read)
  - `actions` (read)

```bash
export GITHUB_TOKEN=ghp_...
Profile Configuration
yaml
Copy
Edit
modules:
  - ci_cd

ci_cd:
  platform: github
  repo: yourorg/yourrepo
  checks:
    job_durations: true
    flakiness: true
    latency: true
    branch_protection: true
    bot_activity: true
    artifact_trends: true
    secrets_usage: true
  limits:
    max_runs: 50      # number of recent workflow runs to analyze
    max_jobs: 500     # safety limit for job iteration
Checks (Detailed)
1) Job Durations
Signals:

Longest-running job in recent runs.

Average duration per workflow/job.

Severity:

Critical if job exceeds set SLO (configurable).

High if average exceeds expected range.

2) Flakiness
Signals:

Jobs that fail intermittently (pass on re-run).

Failure rate percentage > threshold.

Severity:

Failure rate > 20%: high

Failure rate 10–20%: medium

3) Commit-to-Deploy Latency
Signals:

Time from commit push to successful deployment run.

Flags outliers exceeding set threshold.

Severity:

2 hours in prod pipelines: high

30m–2h: medium

4) Branch Protection
Signals:

Missing required reviews.

Force pushes allowed on protected branches.

Missing status checks.

Severity:

Missing branch protection on main/master: critical

Missing status checks: high

5) Bot Activity
Signals:

Jobs triggered by bots (e.g., dependabot[bot]).

Highlights potential automated changes bypassing review.

Severity:

Bot commits without required checks: high

Benign bot runs: low

6) Artifact Trends
Signals:

Build artifacts growing unusually in size.

Flags potential bloat or dependency creep.

Severity:

25% growth between runs: medium

7) Secrets Usage
Signals:

Secrets referenced in workflow YAMLs (${{ secrets.* }}).

Flags high-risk usage in untrusted contexts.

Severity:

Secrets used in fork PR workflows: critical

Secrets used in safe branches: low

Output Schema
json
Copy
Edit
{
  "plugin": "ci_cd",
  "status": "high",
  "messages": [
    "Workflow build.yml job 'test' average runtime is 24m (SLO: 15m)"
  ],
  "metadata": {
    "workflow": "build.yml",
    "job": "test",
    "avg_duration_min": 24,
    "threshold_min": 15
  }
}
Example Findings
Slow Job
csharp
Copy
Edit
[HIGH][ci_cd] Workflow build.yml job 'test' average runtime is 24m (SLO: 15m)
Flaky Job
csharp
Copy
Edit
[HIGH][ci_cd] Job 'deploy' failed in 4 of last 15 runs (26% failure rate)
Missing Branch Protection
csharp
Copy
Edit
[CRITICAL][ci_cd] Branch 'main' is not protected by required status checks
Execution Flow
Authenticate to GitHub API with provided token.

Fetch workflow runs for the repo.

Fetch job details for each run (respecting max_jobs limit).

Run checks for durations, flakiness, latency.

Fetch branch protection settings via REST API.

Inspect workflow files in default branch for secrets usage.

Emit AuditResult list.

Performance & Limits
Limited by GitHub API rate limits (~5000 requests/hour).

SnapCheck batches job fetches to reduce API calls.

max_runs and max_jobs help control API usage.

Security Notes
Token should be read-only and scoped to the repo under audit.

Avoid using a PAT with write/admin scopes unless necessary for other modules.

Troubleshooting
Symptom	Cause	Resolution
403 Forbidden	Missing token scope	Add repo and actions:read scopes
Rate limit exceeded	Too many runs/jobs requested	Lower max_runs or wait for rate limit reset
Branch protection not detected	Repo default branch misconfigured	Confirm default branch in repo settings

Examples
Audit Default GitHub Repo
bash
Copy
Edit
snapcheck run audit --modules ci_cd --profile profiles/prod.yaml
Profile:

yaml
Copy
Edit
ci_cd:
  platform: github
  repo: yourorg/yourrepo
  checks:
    job_durations: true
    flakiness: true
    branch_protection: true
Related Docs
CLI: snapcheck run

Architecture

Security