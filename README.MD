# PodPulse Helm Charts

Official Helm charts for [PodPulse](https://github.com/PodPulse/podpulse-agent) — AI-powered Kubernetes incident diagnostics.

---

## Charts

| Chart | Description |
|---|---|
| `podpulse-agent` | In-cluster agent — detects incidents and forwards them to the PodPulse backend |

---

## Prerequisites

- Kubernetes 1.24+
- Helm 3.10+
- A PodPulse account and API key ([get access](https://github.com/PodPulse/podpulse-agent))

---

## Installation

### 1. Add the PodPulse Helm repository

```bash
helm repo add podpulse https://podpulse.github.io/podpulse-helm
helm repo update
```

### 2. Install the agent

```bash
helm install podpulse-agent podpulse/podpulse-agent \
  --namespace podpulse \
  --create-namespace \
  --set agent.backend.address=api.podpulse.io:443 \
  --set agent.backend.apiKey=pk_live_...
```

Your API key and backend address are provided after installing the PodPulse GitHub App.

---

## Configuration

All parameters can be set via `--set` or a custom `values.yaml` file.

| Parameter | Description | Default |
|---|---|---|
| `agent.backend.address` | PodPulse backend gRPC address | `""` (**required**) |
| `agent.backend.apiKey` | Per-cluster API key | `""` (**required**) |
| `agent.image.repository` | Docker image repository | `ghcr.io/podpulse/podpulse-agent` |
| `agent.image.tag` | Docker image tag | `v0.1.0` |
| `agent.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `agent.replicaCount` | Number of replicas — keep at 1 | `1` |
| `agent.resources.limits.memory` | Memory limit | `64Mi` |
| `agent.resources.limits.cpu` | CPU limit | `100m` |
| `agent.resources.requests.memory` | Memory request | `32Mi` |
| `agent.resources.requests.cpu` | CPU request | `50m` |
| `agent.securityContext.runAsNonRoot` | Run as non-root | `true` |
| `agent.securityContext.runAsUser` | UID | `65532` |
| `agent.securityContext.runAsGroup` | GID | `65532` |
| `agent.securityContext.readOnlyRootFilesystem` | Read-only root filesystem | `true` |
| `serviceAccount.name` | ServiceAccount name | `podpulse-agent` |
| `namespace` | Namespace to deploy into | `podpulse` |

### Using a custom values file

```yaml
# my-values.yaml
agent:
  backend:
    address: api.podpulse.io:443
    apiKey: pk_live_...
  resources:
    limits:
      memory: "128Mi"
      cpu: "200m"
```

```bash
helm install podpulse-agent podpulse/podpulse-agent \
  --namespace podpulse \
  --create-namespace \
  -f my-values.yaml
```

> **Never commit your API key.** Use a secrets manager or pass it at install time via `--set`.

---

## Upgrade

```bash
helm repo update
helm upgrade podpulse-agent podpulse/podpulse-agent \
  --namespace podpulse \
  --set agent.backend.address=api.podpulse.io:443 \
  --set agent.backend.apiKey=pk_live_...
```

The agent pod restarts automatically when credentials change.

---

## Uninstall

```bash
helm uninstall podpulse-agent --namespace podpulse
kubectl delete namespace podpulse
```

The `ClusterRole` and `ClusterRoleBinding` are cluster-scoped and removed automatically by Helm.

---

## Troubleshooting

**Agent pod not starting**

```bash
kubectl describe pod -n podpulse -l app.kubernetes.io/name=podpulse-agent
kubectl logs -n podpulse deployment/podpulse-agent
```

**Incidents not detected**

```bash
# Verify the agent is watching your cluster
kubectl logs -n podpulse deployment/podpulse-agent

# Expected output:
# [INFO] PodPulse agent started — backend: api.podpulse.io:443
```

**Connection refused to backend**

```bash
# Verify the backend address is reachable from the cluster
kubectl run debug --rm -it --image=alpine --restart=Never -- \
  nc -zv api.podpulse.io 443
```

**Invalid API key**

The agent logs `code = Unauthenticated` — verify your API key is correct and not revoked.

---

## Security

The agent runs with minimal permissions:

- **Read-only** Kubernetes access — it never modifies cluster resources
- **No Kubernetes Secrets access** — environment variables and secrets are never collected
- **Distroless runtime** — no shell, no package manager
- **Non-root** — runs as UID 65532
- **Read-only root filesystem**

---

## License

Apache 2.0 — see [LICENSE](https://github.com/PodPulse/podpulse-agent/blob/main/LICENSE)