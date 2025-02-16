# Homelab Infrastructure

This repo contains all of the configuration and documentation of my homelab.


The purpose of my homelab is to learn and to have fun. Being a DevOps Engineer by day, I work with Kubernetes every day, and my homelab is the place where I can try out and learn new things. On the other hand, by self-hosting some applications, it makes me feel responsible for the entire process of deploying and maintaining an application from A to Z. It forces me to think about backup strategies, security, scalability and the ease of deployment and maintenance.

## Cluster Provisioning

I use [Talos Linux](https://www.talos.dev/) for Kubernetes, because it's secure, immutable, and minimal.


## Features

- [Talos](https://www.talos.dev) bare-metal K8s OS
- Lots of [self-hosted services](./kubernetes/main/apps)
- [Flux](https://toolkit.fluxcd.io/) GitOps with this repository ([kubernetes directory](./kubernetes))
- [MetalLB layer 4 loadbalancing](https://metallb.io/concepts/layer2/) 
- [SOPS](https://github.com/mozilla/sops) secrets stored in Git
- [Cloudflared HTTP tunnel](https://github.com/cloudflare/cloudflared)
- [K8s gateway](https://github.com/ori-edge/k8s_gateway) for local DNS resolution to the cluster and [NGINX ingress controller](https://kubernetes.github.io/ingress-nginx/)
- Both internal & external services with a service [gateway](https://github.com/ori-edge/k8s_gateway/)
- Automatic Cloudflare DNS updates with [external-dns](./kubernetes/main/apps/network/external-dns/app/helmrelease.yaml)
- [CloudNative-PG](https://cloudnative-pg.io/) with automatic failover
- [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) with various Grafana dashboards
- [Longhorn](https://longhorn.io/) cluster storage

## Mainfraime Configuration

### Node Details

- **Node Name**: `control-plane-1`
- **Model**:  RasberryPi 4 B
- **Specifications**:
  - **CPU**: Cortex-A72 4 CPU Cores
  - **RAM**: 8 GB
  - **Storage**: 240 GB NVMe

---

- **Node Name**: `worker-1`
- **Model**:  RasberryPi 4 B
- **Specifications**:
  - **CPU**: Cortex-A72 4 CPU Cores
  - **RAM**: 8 GB
  - **Storage**: 240 GB NVMe

----

- **Node Name**: `worker-2`
- **Model**:  EliteDesk 800 G4 Desktop Mini
- **Specifications**:
  - **CPU**: i5-4800 6 CPU Cores
  - **RAM**: 32 GB
  - **Storage**: 500 GB NVMe

#### Setup FluxCD

Bootstrap Flux

```sh
kubectl apply --server-side --kustomize kubernetes/main/flux-system/app.yaml
```

Add SOPS key to Flux
```sh
kubectl create secret generic sops-age \
  --namespace=flux-system \ 
  --from-file=danielnikoloski_sops.agekey
```

#### DNS and Tunnel

Setup a Cloudflare Tunnel.

```sh
cloudflared tunnel login
cloudflared tunnel create cluster
```

Add the tunnel's `credentials.json` to the value in [`cloudflared-secret`](kubernetes/apps/network/cloudflared/app/secret.sops.yaml) and tunnel ID to `cluster-secrets.sops.yaml`.

Add a Cloudflare API token with these permissions to the value in [`external-dns-secret`](kubernetes/apps/network/external-dns/app/secret.sops.yaml).

- `Zone - DNS - Edit`
- `Zone - Zone - Edit`
- `Account - Cloudflare Tunnel - Read`

### Directories

This Git repository contains the following directories under [Kubernetes](./kubernetes/). Check out [cluster-template](https://github.com/onedr0p/flux-cluster-template) for more details on how this FluxCD setup works.

```sh
📁 kubernetes
├── 📁 main # main cluster
│   ├── 📁 apps # applications
│   ├── 📁 flux # core flux configuration
└── 📁 ...
```

### Storage

Upgrade Talos nodes with custom Extensions in order to make Longhorn work

- Create a new image -
[Talos Linux Image Factory](https://factory.talos.de)
```yaml
customization:
  systemExtensions:
    officialExtensions:
      - siderolabs/iscsi-tools
      - siderolabs/util-linux-tools
```

Upgrade the node (example)
```sh
talosctl upgrade --image factory.talos.dev/installer/f8a903f101ce10f686476024898734bb6b36353cc4d41f348514db9004ec0a9d:v1.9.4 -n 10.0.10.20
```

Edit machine and add Data Path Mounts

```yaml
machine:
  kubelet:
    extraMounts:
      - destination: /var/lib/longhorn
        type: bind
        source: /var/lib/longhorn
        options:
          - bind
          - rshared
          - rw
```
