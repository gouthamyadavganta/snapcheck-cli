# SnapCheck Documentation

**Version:** v1.0  
**Status:** Pilot-ready

---

## Overview

SnapCheck is an **enterprise-grade, modular auditing and correlation platform** for DevOps and MLOps environments.  
It unifies infrastructure, deployment, security, and cost signals into a single, repeatable, and shareable view.

Designed for **SRE, Platform, and Security teams**, SnapCheck runs read-only audits across:

- **Terraform** (state drift, IAM policy risks, public exposure, stale resources, cost estimation)
- **Kubernetes** (node/pod health, PVC/DNS/service checks, baseline security)
- **Helm** (release inventory, failed upgrades, outdated charts, values drift)
- **CI/CD (GitHub)** (job metrics, flakiness, branch protection, artifact trends)
- **Docker registries** (tag listing, metadata, CVE lookup)
- **Secrets** (inventory, regex leak detection, secret age, unused detection)
- **AWS Cost Explorer** (real monthly cost, service breakdown, cross-check vs Terraform inventory)
- **GitOps (Argo CD)** (health, sync status, revision drift, auto-sync flags)

SnapCheck **correlates** these findings into actionable storylines:
> _Example: Terraform drift → Helm upgrade failure → CI latency spike → AWS cost anomaly_

---

## Key Features

- **Modular Plugins** — Enable only what you need per environment.
- **Correlation Engine** — Connects root causes with downstream symptoms.
- **Multi-Format Output** — Terminal, Markdown, and HTML dashboard.
- **History & Regression Tracking** — Every run is timestamped, with diffs available.
- **Secure by Default** — GitHub OAuth, RBAC, secrets from env/vault, no plaintext storage.
- **Shareable Reports** — Publish to GitHub Pages or export as static HTML.

---

## Typical Use Cases

- **Incident Investigation**
  - Reduce mean-time-to-understand (MTTU) by combining cross-system events into a single storyline.
- **Compliance & Audit Prep**
  - Generate portable HTML/Markdown reports for SOC 2, ISO 27001, and internal reviews.
- **Cost & Resource Hygiene**
  - Identify unused resources, secret leaks, and unexpected spend changes.
- **Release Readiness**
  - Assess infra health, drift, and pipeline posture before high-risk deployments.

---

## Audience

SnapCheck is built for:
- **Site Reliability Engineers (SRE)**
- **Platform/DevOps Engineers**
- **Cloud Security Teams**
- **Leadership & Compliance Officers**

---

## Documentation Structure

- [Getting Started](getting-started.md) — Installation, profile setup, first audit run.
- [Architecture](architecture.md) — System components and data flow.
- [Security](security.md) — Authentication, RBAC, secrets handling.
- [Operations Guide](operations.md) — Running audits, handling failures, DR strategy.
- [CLI Reference](cli/run.md) — Detailed command usage.
- [Plugin Reference](plugins/terraform.md) — Checks and outputs per plugin.
- [Developer Guide](developer/correlation.md) — Extending SnapCheck.

---

## Quick Links

- **GitHub Repository:** [<YOUR_REPO_URL>](<YOUR_REPO_URL>)
- **CLI Help:** `snapcheck --help`
- **License:** MIT (see LICENSE file)

---
