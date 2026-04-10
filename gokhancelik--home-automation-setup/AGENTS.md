# Copilot Instructions

## Project Overview
This repository is a **GitOps-managed Kubernetes Home Assistant platform** using **Flux CD**. It runs on a mixed architecture MicroK8s cluster (Ubuntu Desktop Master + Raspberry Pi Worker).

## Operational Commands

### GitOps & Flux
Changes are applied via git commits.
- **Reconcile**: `flux reconcile kustomization flux-system --with-source`
- **Check Status**: `flux get kustomizations` or `flux get helmreleases -A`
- **Logs**: `flux logs -f`

### Kubernetes Debugging
- **Pods**: `kubectl get pods -n <namespace>`
- **Logs**: `kubectl logs -n <namespace> <pod_name>`
- **Events**: `kubectl get events -n <namespace> --sort-by='.lastTimestamp'`
- **Shell**: `kubectl exec -it -n <namespace> <pod_name> -- /bin/bash`

### Secrets Management
Secrets must be encrypted using Sealed Secrets.
- **Seal a secret**:
  ```bash
  kubectl create secret generic <name> --namespace <ns> --from-literal=key=value --dry-run=client -o yaml | \
  kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system --format=yaml > sealed-secret.yaml
  ```

## Architecture

### Directory Structure
- `clusters/microk8s/`: The cluster definition and entry point.
- `charts/`: Local Helm charts (e.g., `home-assistant`).
- `infrastructure/`: Core services (cert-manager, ingress, observability).
- `databases/`: Stateful services (PostgreSQL, InfluxDB).
- `apps/`: Application workloads (Home Assistant, Frigate).

### Infrastructure
- **Nodes**:
  - `ubuntu-desktop` (Master): Control Plane, DBs, Frigate.
  - `pi-lab` (Worker): Zigbee/Matter USB Coordinators.
- **Storage**:
  - `nfs-storage`: NAS-backed PVCs (Default for stateful apps).
  - `ssd-storage`: Local SSD (Avoid for persistent data).

## Key Conventions

### Development Rules
- **GitOps Only**: Do not use `kubectl apply` to make changes. Edit manifests, commit, and let Flux sync.
- **No HA Supervisor**: This is a "Core" installation. Do not suggest Add-ons or Supervisor features.
- **Database**:
  - **PostgreSQL**: MANDATORY for Home Assistant recorder. No SQLite.
  - **InfluxDB**: Used for long-term metrics.
- **Hardware Isolation**:
  - Zigbee/Matter radios are physically attached to the Pi (`pi-lab`).
  - Use `nodeSelector` or `affinity` to pin USB-dependent pods.

### Manifest Guidelines
- **Version Pinning**: All Helm charts and container images must have explicit versions (no `latest`).
- **Resources**: All pods must have `requests` and `limits` defined.
- **Secrets**: Never commit raw secrets. Use `SealedSecret` resources.
- **Namespace Layout**:
  - `home-automation`: Home Assistant
  - `data-storage`: PostgreSQL, InfluxDB
  - `observability`: Loki, Grafana, Promtail
  - `zigbee`: Zigbee2MQTT, Mosquitto
  - `frigate`: Frigate NVR

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gokhancelik)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gokhancelik)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
