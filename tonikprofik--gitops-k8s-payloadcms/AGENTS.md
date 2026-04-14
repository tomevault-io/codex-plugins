# gitops-k8s-payloadcms Copilot Instructions

## Project Purpose

GitOps reference architecture for deploying **PayloadCMS** (TypeScript/React/Next.js headless CMS) 
to Kubernetes. A hands on learning project on SRE/Platform Engineering skills.

## Architecture Overview

GitOps-driven Kubernetes platform using **ArgoCD App-of-Apps pattern**:
- `argocd/bootstrap.yaml` → single entrypoint deploying all apps via `argocd/applications/`
- Sync waves control deployment order: observability (-1) → payload-cms (1)
- Two deployment targets: **k3d (local)** and **Hetzner Cloud (production)**

## Directory Structure

```
pubdocs/                 # 📖 Deep-dive documentation
├── ARCHITECTURE.md      # System design + multi-cloud + Talos roadmap
└── QUICKSTART.md        # Setup guide + gotchas + production checklist

apps/                    # Kubernetes manifests (Kustomize + Helm)
├── observability/       # Prometheus, Loki (Helm charts)
└── payload-cms/         # PayloadCMS + Postgres (raw manifests)

argocd/
├── bootstrap.yaml       # App-of-Apps root (apply once)
└── applications/        # ArgoCD Application CRDs

infra/
├── hetzner/            # Terraform for VPS + k3s
├── aws/                # AWS EKS (planned, README only)
└── talos/              # Talos Linux (planned)
    ├── bare-metal/     # For homelab/physical servers
    └── hetzner-dedicated/  # For Hetzner Robot servers
```

## Developer Workflow

### Local Development (k3d)
```bash
make local-up     # Creates k3d cluster + installs ArgoCD + applies bootstrap
make local-down   # Destroys cluster
```

### Production (Hetzner)
```bash
make hetzner-up        # Terraform provision VPS with k3s via cloud-init
make hetzner-kubeconfig # Fetch kubeconfig from remote
make hetzner-destroy    # Tear down infrastructure
```

### Access Services
```bash
make argocd-password       # Get ArgoCD admin password
make port-forward-argocd   # localhost:8080 → ArgoCD UI
make port-forward-grafana  # localhost:3000 → Grafana
```

## Key Patterns

### Adding New Applications
1. Create manifests in `apps/<app-name>/` with `kustomization.yaml`
2. Add ArgoCD Application in `argocd/applications/<app-name>.yaml`
3. Set `sync-wave` annotation for deployment order
4. ArgoCD auto-syncs from Git (prune + selfHeal enabled)

### Helm in Kustomize
Observability stack uses `helmCharts` in Kustomization:
```yaml
helmCharts:
  - name: kube-prometheus-stack
    repo: https://prometheus-community.github.io/helm-charts
    valuesFile: values-prometheus.yaml
```

### Secrets Management
Current: Kustomize `secretGenerator` (demo only)
Production: Use SealedSecrets or ExternalSecrets Operator

## Infrastructure Notes

- **Hetzner VPS**: `cpx11` (2 vCPU, 2GB) with k3s installed via `user_data.yml`
- **k3d local**: 2 agents, port 8080 exposed for LoadBalancer
- PayloadCMS expects `payload.localhost:8080` (add to `/etc/hosts`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tonikprofik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
