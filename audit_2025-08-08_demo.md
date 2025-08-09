# SnapCheck Audit Report — demo
_Audit Timestamp: 2025-08-08 21:24:52_

## Modules Summary
| Module | Status |
|---|---|
| terraform | ℹ️ INFO |
| kubernetes | ℹ️ INFO |
| helm | ℹ️ INFO |
| ci_cd | ℹ️ INFO |
| docker | ℹ️ INFO |
| secrets | ℹ️ INFO |
| cost | ℹ️ INFO |
| gitops | ℹ️ INFO |
| correlation | ℹ️ INFO |

## Module Details
### Terraform — ℹ️ INFO

- **[INFO]** ℹ️ **Terraform**
  ```
  {'title': 'Terraform', 'status': '✅ OK', 'messages': ['No drift detected in last snapshot.', 'State age: 2 days.']}
  ```

### Kubernetes — ℹ️ INFO

- **[INFO]** ℹ️ **Kubernetes**
  ```
  {'title': 'Kubernetes', 'status': '🟡 Warning', 'messages': ['2 pods restarting (CrashLoopBackOff) in monitoring namespace.', 'Node pressure: none. DNS: OK.']}
  ```

### Helm — ℹ️ INFO

- **[INFO]** ℹ️ **Helm**
  ```
  {'title': 'Helm', 'status': '✅ OK', 'messages': ['All releases healthy. 1 chart outdated (minor).'], 'metadata': {'outdated': [{'name': 'prometheus', 'current': '19.6.0', 'latest': '19.7.1'}]}}
  ```

### Ci Cd — ℹ️ INFO

- **[INFO]** ℹ️ **Ci Cd**
  ```
  {'title': 'CI/CD', 'status': '✅ OK', 'messages': ['Avg durations — Build: 22s, Test: 41s, Deploy: 19s.', 'Branch protection: enabled on main.']}
  ```

### Docker — ℹ️ INFO

- **[INFO]** ℹ️ **Docker**
  ```
  {'title': 'Docker', 'status': '✅ OK', 'messages': ['2 images scanned via registry API. No critical CVEs.', 'Largest image: 412 MB.']}
  ```

### Secrets — ℹ️ INFO

- **[INFO]** ℹ️ **Secrets**
  ```
  {'title': 'Secrets', 'status': '✅ OK', 'messages': ['No high-risk hardcoded secrets detected in tracked paths.', 'Kubernetes secrets: 12 (10 used, 2 unused).']}
  ```

### Cost — ℹ️ INFO

- **[INFO]** ℹ️ **Cost**
  ```
  {'title': 'Cost', 'status': '✅ OK', 'messages': ['Monthly AWS cost (last 4 months): $120 → $150 → $90 → $170.'], 'metadata': {'monthly_cost_usd': [120, 150, 90, 170]}}
  ```

### Gitops — ℹ️ INFO

- **[INFO]** ℹ️ **Gitops**
  ```
  {'title': 'GitOps', 'status': '✅ OK', 'messages': ['ArgoCD apps in sync ✅  Self-heal: enabled.']}
  ```

### Correlation — ℹ️ INFO

- **[INFO]** ℹ️ **Correlation**
  ```
  {'title': 'Root Cause Correlation', 'status': '✅ OK', 'messages': ['Potential link: image size growth ↔ longer build times.', 'No incident linkage detected in last 24h.']}
  ```
