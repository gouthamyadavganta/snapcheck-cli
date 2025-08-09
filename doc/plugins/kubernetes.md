# Kubernetes Plugin

The Kubernetes plugin audits the **health, availability, and baseline security posture** of a Kubernetes cluster.  
It focuses on **node and pod health**, **critical workload readiness**, **storage and DNS issues**, and **basic security context checks**.

---

## Capabilities (v1.0)

- **Node health checks**: Readiness, pressure conditions, schedulability.
- **Pod health**: CrashLoopBackOff, ImagePullBackOff, excessive restarts.
- **Probe failures**: Liveness/readiness probe failures.
- **Storage**: PVC pending/binding issues.
- **Networking**: CoreDNS resolution failures, missing service endpoints.
- **Security contexts**: Privileged pods, hostPath volumes, dangerous capabilities.
- **Restart count tracking**: Detect workloads with high or increasing restart counts.

---

## Prerequisites

- Access to the target cluster via `kubeconfig`:
  - Default: `$HOME/.kube/config`
  - Override in profile: `kubeconfig: /path/to/config`
- RBAC permissions:
  - `get`, `list`, `watch` on:
    - `nodes`, `pods`, `services`, `persistentvolumeclaims`
  - Read access to `kube-system` namespace for CoreDNS checks.

> The plugin **does not** modify resources or apply manifests.

---

## Profile Configuration

```yaml
modules:
  - kubernetes

kubernetes:
  kubeconfig: ~/.kube/config
  namespaces: [default, kube-system, monitoring]   # If omitted, audits all namespaces
  checks:
    node_health: true
    pod_health: true
    probes: true
    pvc: true
    dns: true
    services: true
    security_contexts: true
    restarts: true
  limits:
    max_pods: 5000   # hard stop for very large clusters
Checks (Detailed)
1) Node Health
Signals:

NotReady condition

MemoryPressure, DiskPressure, PIDPressure

unschedulable: true taint

Severity:

NotReady node: critical

Pressure condition: high

Unschedulable: medium (unless intentional)

2) Pod Health
Signals:

CrashLoopBackOff, ImagePullBackOff, ErrImagePull, CreateContainerConfigError

Pods stuck in Pending > 5 minutes

Severity:

CrashLoop on critical namespace (e.g., kube-system): critical

CrashLoop on app namespace: high

Image pull errors: high

3) Probes
Signals:

Liveness/readiness probes failing consistently

Readiness probe never succeeded after pod start

Severity:

Liveness failure (restarts pod): high

Readiness never succeeded: medium

4) Persistent Volume Claims (PVC)
Signals:

PVC stuck in Pending state > threshold (default: 5 minutes)

PVC bound to non-existing PV

Severity:

PVC unbound in critical namespace: high

PVC unbound in app namespace: medium

5) DNS Resolution
Signals:

CoreDNS pod not ready

CoreDNS crash looping

Unable to resolve default service (kubernetes.default.svc)

Severity:

CoreDNS down: critical

CoreDNS unstable: high

Single DNS query fail: medium

6) Service Endpoint Reachability
Signals:

Services with no ready endpoints

Endpoints not matching selector pods

Severity:

No endpoints in critical namespace: high

No endpoints in app namespace: medium

7) Security Contexts
Signals:

securityContext.privileged: true

hostPath volumes mounted

Dangerous capabilities (NET_ADMIN, SYS_ADMIN, etc.)

Severity:

Privileged pod in prod: critical

Privileged pod in non-prod: high

HostPath volume: high

Dangerous capability: medium/high depending on context

8) Restarts
Signals:

Pod restart count exceeds threshold (default: 5)

Increase in restart count since last run

Severity:

High restart count on critical pod: high

High restart count on app pod: medium

Output Schema
json
Copy
Edit
{
  "plugin": "kubernetes",
  "status": "critical",
  "messages": [
    "Node ip-10-0-3-12 NotReady (DiskPressure)"
  ],
  "metadata": {
    "resource": "node/ip-10-0-3-12",
    "category": "node_health",
    "namespace": null
  }
}
Example Findings
Node Down
scss
Copy
Edit
[CRITICAL][kubernetes] Node ip-10-0-3-12 NotReady (DiskPressure)
Pod CrashLoop
less
Copy
Edit
[HIGH][kubernetes] Pod payments-api-6d8f6ddf4-abcde CrashLoopBackOff (restarts: 12)
PVC Pending
csharp
Copy
Edit
[MEDIUM][kubernetes] PVC logs-storage Pending > 5m in namespace staging
Privileged Pod
scss
Copy
Edit
[CRITICAL][kubernetes] Pod debug-tools in namespace default runs privileged
Execution Flow
Connect to cluster via kubeconfig.

List nodes and check conditions.

List pods (optionally namespace-filtered) and check:

Status

Restart counts

Probes

List PVCs and check binding state.

Check DNS by inspecting CoreDNS pod readiness and optionally running an in-cluster DNS resolution test.

Check services for ready endpoints.

Check security contexts for privileged flags, hostPath, capabilities.

Emit AuditResult list.

Performance & Limits
Defaults to all namespaces; filter via namespaces in profile for faster runs.

limits.max_pods prevents runaway audits on massive clusters.

DNS check can be disabled if CoreDNS pods are intentionally absent (e.g., custom DNS).

Security Notes
Requires read-only Kubernetes RBAC.

Recommended minimal RBAC role:

yaml
Copy
Edit
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: snapcheck-readonly
rules:
  - apiGroups: [""]
    resources: ["nodes","pods","services","persistentvolumeclaims"]
    verbs: ["get","list","watch"]
Troubleshooting
Symptom	Cause	Resolution
Error loading kubeconfig	Wrong path or missing file	Update kubeconfig path in profile
Forbidden errors	Missing RBAC permissions	Apply read-only ClusterRoleBinding
DNS check fails	CoreDNS intentionally replaced/absent	Disable dns check in profile
Service endpoint check false positives	Headless or intentionally empty service	Add exceptions or disable services check

Examples
Audit All Namespaces
bash
Copy
Edit
snapcheck run audit --modules kubernetes --profile profiles/prod.yaml
Audit Only Critical Namespaces
yaml
Copy
Edit
kubernetes:
  kubeconfig: ~/.kube/config
  namespaces:
    - kube-system
    - monitoring
Related Docs
CLI: snapcheck run

Architecture

Security

Developer: Correlation Engine