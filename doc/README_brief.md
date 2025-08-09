# 🚀 SnapCheck — One-Pass DevOps/MLOps Audit & Correlation

> **A single CLI + web UI** that audits Terraform, Kubernetes, Helm, CI/CD, Docker, Secrets, AWS Cost, and GitOps — then **connects the dots** so you know *what broke, why, and what changed*.

---

## 📌 Why SnapCheck

- **Too many tools, too little context** → SnapCheck pulls from all your DevOps/MLOps systems at once.
- **Root cause clarity** → Links Terraform drift → Helm failure → CI delay → Cost spike.
- **Audit painkiller** → Portable HTML/Markdown reports ready for compliance or reviews.
- **Security-first** → OAuth/RBAC, read-only access, no secrets stored in-app.

---

## 🔍 What It Audits (v1)

| Plugin       | Key Checks |
|--------------|-----------|
| **Terraform** | Drift, IAM issues, public S3/SG, stale resources |
| **Kubernetes** | Node readiness, pod health, PVC binding, DNS/services |
| **Helm** | Failed releases, outdated charts, values drift |
| **CI/CD (GitHub)** | Longest jobs, flakiness, branch protection gaps |
| **Docker** | Remote registry scan, CVEs, tag metadata |
| **Secrets** | GitHub/K8s secret inventory, leaks, age |
| **Cost (AWS)** | Monthly spend by service, Terraform-linked inventory |
| **GitOps (Argo CD)** | App sync/health, drift, failed syncs |

---

## 🖥 Output & Access

- **Terminal:** Quick results for engineers.
- **Markdown:** Shareable in PRs or wikis.
- **HTML Dashboard:** Tailwind + Chart.js — KPIs, severity highlights, history.
- **Web UI:** GitHub OAuth + RBAC, expiring share links.

---

## 🛠 Quick Start

```bash
# Install
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Create profile
snapcheck init-profile --init-name prod --init-output profiles/prod.yaml --quickstart

# Run audit
snapcheck run audit --modules all --output terminal

# View dashboard
snapcheck serve
📚 Learn More
First User Guide: docs/first-user-guide.md

Full Technical Docs: docs/

Security Model: docs/security.md

📊 Architecture Overview
mermaid
Copy
Edit
flowchart LR
    CLI["snapcheck run"] --> Plugins
    Plugins --> Correlator["Correlation Engine"]
    Correlator --> Reports["HTML / Markdown / History"]
    Reports -->|serve| WebUI["FastAPI Web UI"]
    WebUI -->|OAuth/RBAC| User
💡 Pilot-ready now — run in your environment, get full system health in one pass