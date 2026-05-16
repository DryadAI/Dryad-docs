# Dryad Phase 7 — Talos Kubernetes Cluster Bootstrap Plan

> **Status:** PLANNING  
> **Node:** madhatter (initial control plane)  
> **Last updated:** 2026-04-13

---

## Overview

Phase 7 migrates Project Dryad from Docker Compose on a single host to a Kubernetes cluster managed by Talos Linux. Goals:

- **Multi-node support** — add compute nodes without reconfiguring services
- **GPU workload scheduling** — NVIDIA device plugin for RTX 3060 + future GPUs
- **Persistent storage** — Longhorn or local-path provisioner for AI data volumes
- **Rolling updates** — zero-downtime deploys for Dryad services
- **Declarative infra** — all config in git, applied via `kubectl` / Helm

---

## Why Talos

| Feature | Benefit |
|---------|---------|
| Immutable OS | No SSH, no shell — attack surface minimal |
| API-driven config | Machine configs in git, applied via `talosctl` |
| Auto-upgrades | Kubernetes and OS upgrades via single API call |
| Minimal footprint | ~180MB base OS; no package manager |
| Arch Linux compatible | Talos runs as VM or bare-metal; coexists with Arch |

---

## Cluster Design

### Initial Topology (Phase 7.0)

```
madhatter (Arch Linux host)
├── talos-cp-01 (VM or bare-metal partition)
│   └── Role: control-plane + worker
│   └── GPU: RTX 3060 passed through / direct
│   └── RAM allocation: 12GB to cluster
│   └── CPU: 6 cores
└── talos-w-01 (optional second worker — same machine, different node)
```

Single-node initially. Add physical nodes later via same config pattern.

### Future Topology (Phase 7.x)

```
talos-cp-01  (madhatter)         — control plane
talos-w-01   (madhatter)         — GPU worker
talos-w-02   (future node)       — CPU worker
talos-w-03   (future node)       — storage worker
```

---

## Prerequisites

- [ ] Talos Linux ISO downloaded
- [ ] `talosctl` installed on madhatter
- [ ] `kubectl` installed on madhatter
- [ ] NVIDIA container toolkit available for Talos (check extensions)
- [ ] Static IP or DHCP reservation for cluster nodes
- [ ] Storage decision: Longhorn vs local-path provisioner
- [ ] Tailscale subnet routing for cluster services (or Cilium CNI with Tailscale)

---

## Bootstrap Steps

### Step 1 — Generate Talos Config

```bash
# Generate secrets + cluster config
talosctl gen config dryad-cluster https://<control-plane-ip>:6443 \
  --config-patch @patches/gpu-patch.yaml \
  --output-dir ./talos-config

# Files generated:
#   talos-config/controlplane.yaml
#   talos-config/worker.yaml
#   talos-config/talosconfig
```

### Step 2 — Apply Config to Node

```bash
# Boot node from Talos ISO, then:
talosctl apply-config \
  --nodes <node-ip> \
  --file talos-config/controlplane.yaml \
  --insecure
```

### Step 3 — Bootstrap Cluster

```bash
talosctl bootstrap --nodes <control-plane-ip> \
  --talosconfig talos-config/talosconfig
```

### Step 4 — Get kubeconfig

```bash
talosctl kubeconfig \
  --nodes <control-plane-ip> \
  --talosconfig talos-config/talosconfig \
  ~/.kube/config
```

### Step 5 — Verify Cluster

```bash
kubectl get nodes
kubectl get pods -A
```

---

## GPU Configuration

### Talos Machine Config Patch (GPU)

```yaml
# patches/gpu-patch.yaml
machine:
  kernel:
    modules:
      - name: nvidia
      - name: nvidia_uvm
      - name: nvidia_drm
      - name: nvidia_modeset
  sysctls:
    net.core.rmem_max: "2500000"
cluster:
  externalCloudProvider:
    enabled: false
```

### NVIDIA Device Plugin

```bash
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.0/nvidia-device-plugin.yml
```

### Verify GPU Available

```bash
kubectl describe node | grep nvidia.com/gpu
```

---

## Storage

### Option A — local-path-provisioner (simple)

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Option B — Longhorn (distributed, HA)

```bash
helm repo add longhorn https://charts.longhorn.io
helm install longhorn longhorn/longhorn \
  --namespace longhorn-system \
  --create-namespace
```

### AI Data Volume

```yaml
# pvc-ai-engine.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ai-engine-pvc
  namespace: dryad
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: local-path
  resources:
    requests:
      storage: 100Gi
```

---

## Dryad Services Migration Order

| Priority | Service | Notes |
|----------|---------|-------|
| 1 | Redis | Stateful — migrate with PVC |
| 2 | Qdrant | Stateful — migrate with PVC |
| 3 | LiteLLM | Stateless — easy deploy |
| 4 | Ollama | GPU workload — requires device plugin |
| 5 | Open WebUI | Stateless |
| 6 | Dryad Portal | Stateless — Bun container |
| 7 | opencode-spun | Stateless |
| 8 | MCP Station | Stateless |
| 9 | Mem0 | Depends on Redis + Qdrant |
| 10 | OpenWork | Stateless |

---

## Networking

### CNI Options

| CNI | Why |
|-----|-----|
| Flannel | Simple, single-node default |
| Cilium | eBPF, Tailscale integration, NetworkPolicy support |

**Recommendation:** Cilium for Phase 7 — Tailscale integration avoids separate ingress config.

### Ingress

```bash
# Nginx ingress for HTTP routing
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace
```

Caddy replaced by Kubernetes Ingress + cert-manager for TLS.

---

## Namespace Layout

```
dryad/          — all Dryad services
dryad-system/   — operators, monitoring
ingress-nginx/  — ingress controller
longhorn-system/ — storage (if Longhorn)
gpu-operator/   — NVIDIA device plugin
```

---

## Observability

```bash
# Prometheus + Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prometheus prometheus-community/kube-prometheus-stack \
  --namespace dryad-system \
  --create-namespace
```

Replaces Netdata. GPU metrics via `dcgm-exporter`.

---

## Migration Checklist

### Phase 7.0 — Single Node

- [ ] Talos ISO prepared and tested in VM
- [ ] GPU passthrough verified (nvidia-smi inside cluster)
- [ ] local-path-provisioner deployed
- [ ] Redis + Qdrant migrated with data intact
- [ ] LiteLLM running, gateway accessible
- [ ] Ollama running, models loaded
- [ ] Dryad Portal accessible via Ingress
- [ ] log sync timer migrated to CronJob

### Phase 7.1 — Multi-Node

- [ ] Second node provisioned with Talos
- [ ] Longhorn deployed for distributed storage
- [ ] GPU node taint applied (`nvidia.com/gpu=present:NoSchedule`)
- [ ] Ollama + portal scheduled on GPU node
- [ ] Stateless services spread across workers

### Phase 7.2 — Production Hardening

- [ ] cert-manager deployed (Let's Encrypt or self-signed)
- [ ] NetworkPolicy applied to all namespaces
- [ ] Secrets migrated to Kubernetes Secrets (or Sealed Secrets)
- [ ] Backup strategy for PVCs (Velero or manual)
- [ ] Talos automatic upgrade plan configured

---

## dryad-systems Log Sync in Kubernetes

Replace systemd timer with a CronJob:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dryad-log-sync
  namespace: dryad
spec:
  schedule: "0 * * * *"   # every hour
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: sync
            image: alpine/git
            command: ["/scripts/sync-logs.sh"]
            volumeMounts:
            - name: scripts
              mountPath: /scripts
            - name: logs
              mountPath: /logs
          volumes:
          - name: scripts
            configMap:
              name: dryad-scripts
          - name: logs
            persistentVolumeClaim:
              claimName: dryad-logs-pvc
          restartPolicy: OnFailure
```

---

## Notes

- Talos is API-only — no SSH. All node management via `talosctl`.
- Machine configs live in `talos-config/` — **never commit secrets** from that directory.
- RTX 3060 VRAM (12GB) is sufficient for current models. Phase 7 GPU scheduling must account for 16GB host RAM constraint.
- Longhorn requires at least 3 nodes for HA; use local-path for single-node Phase 7.0.
