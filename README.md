# FreeCloud Initiative - Ansible Automation

This repository contains the Ansible automation for provisioning and managing a K3s cluster on Raspberry Pi nodes. It handles everything from initial node preparation and boot configuration to the installation of core Kubernetes services like storage (Longhorn), networking (Cilium), and GitOps (ArgoCD).

## Repository Structure

| Path | Type | Description |
| :--- | :--- | :--- |
| [**roles/**](./roles/) | Directory | Contains all individual Ansible roles for specific tasks and service installations. |
| [**collections/**](./collections/) | Directory | Defines the required Ansible collections (`kubernetes.core`, `community.general`, `ansible.posix`). |
| [**group_vars/**](./group_vars/) | Directory | Global and group-specific variable definitions. |
| [**playbook.yml**](./playbook.yml) | File | The primary playbook for full K3s cluster deployment and service configuration. |
| [**boot_config.yml**](./boot_config.yml) | File | Playbook for configuring Raspberry Pi specific `config.txt` settings. |
| [**inventory.ini**](./inventory.ini) | File | Defines the cluster inventory, grouped by node roles (master/worker) and resources. |
| [**ansible.cfg**](./ansible.cfg) | File | Local Ansible configuration (inventory path, roles path, etc.). |
| [**README.md**](./README.md) | File | This file. |

## Roles Overview

| Role | Description |
| :--- | :--- |
| `k3s-presetup` | Prepares nodes by disabling swap and installing required base packages. |
| `k3s-bootstrap` | Generates a secure random bootstrap token for cluster joining. |
| `k3s-master-setup` | Installs and initializes the K3s server on the master node. |
| `distribute-k3s-node-password` | Distributes the cluster token to worker nodes for join automation. |
| `k3s-worker-setup` | Installs the K3s agent on worker nodes and joins them to the cluster. |
| `cleanup-bootstrap` | Removes temporary bootstrap files once the cluster is initialized. |
| `k3s-node-labeling` | Applies custom Kubernetes labels to nodes (e.g., node tiers based on RAM). |
| `cilium-setup` | Installs Cilium CNI with WireGuard encryption and Hubble enabled. |
| `longhorn-setup` | Installs Longhorn distributed block storage via Helm. |
| `traefik-setup` | Configures and deploys Traefik Ingress controller. |
| `argocd-setup` | Deploys ArgoCD for GitOps management of cluster resources. |
| `sealed-secrets-setup` | Installs Bitnami Sealed Secrets for secure secret management. |
| `kyverno-setup` | Deploys Kyverno for policy-based cluster management. |
| `raspberry-pi-boot-config` | Manages low-level Raspberry Pi hardware configurations. |

## Quick Start

### 1. Install Dependencies
Ensure you have the required collections installed:
```bash
ansible-galaxy collection install -r collections/requirements.yml
```

### 2. Configure Boot Settings
If you are using Raspberry Pis, configure the boot settings first:
```bash
ansible-playbook boot_config.yml
```

### 3. Deploy K3s Cluster
Run the main playbook to setup the entire cluster and all services:
```bash
ansible-playbook playbook.yml
```

## Linting
This project follows `ansible-lint` standards. To check for compliance:
```bash
ansible-lint roles
```

## License
MIT
