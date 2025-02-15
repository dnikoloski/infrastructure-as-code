# Homelab Infrastructure

This repo contains all of the configuration and documentation of my homelab.


The purpose of my homelab is to learn and to have fun. Being a DevOps Engineer by day, I work with Kubernetes every day, and my homelab is the place where I can try out and learn new things. On the other hand, by self-hosting some applications, it makes me feel responsible for the entire process of deploying and maintaining an application from A to Z. It forces me to think about backup strategies, security, scalability and the ease of deployment and maintenance.

## Cluster Provisioning

I use [Talos Linux](https://www.talos.dev/) for Kubernetes, because it's secure, immutable, and minimal.


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

## Current Goals

* Set up Grafana and Prometheus
* Alert notifcation to Telegram/Discord
* Increase security
