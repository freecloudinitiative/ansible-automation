# Ansible Roles Catalog

This directory contains Ansible roles used to provision, configure, and manage the **Free Cloud Initiative** infrastructure. The playbooks orchestrate these roles to configure Raspberry Pi boot settings, set up a K3s Kubernetes cluster, and deploy crucial cloud-native infrastructure components.

## Infrastructure & OS Management Roles

These roles handle initial hardware/OS configuration and node health checks.

| Role                                                   | Target Hosts               | Description                                                                                                                                                                             |
| ------------------------------------------------------ | -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`raspberry-pi-boot-config`](raspberry-pi-boot-config) | `all` / Raspberry Pi nodes | Configures boot options specific to Raspberry Pi nodes, such as enabling container features in `/boot/cmdline.txt` (e.g., `cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory`). |
| [`rpi-thermal-check`](rpi-thermal-check)               | `all` / Raspberry Pi nodes | Monitors and logs CPU temperatures on Raspberry Pi nodes to prevent overheating and thermal throttling.                                                                                 |

---

## K3s Cluster Setup Roles

These roles manage the lifecycle of the K3s Kubernetes cluster, from initial node preparation to joining worker nodes.

| Role                                       | Target Hosts | Description                                                                                                                                                             |
| ------------------------------------------ | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`k3s-pre-setup`](k3s-pre-setup)           | `all`        | Prepares target hosts for K3s installation. Disables swap, configures necessary kernel modules, installs prerequisite system packages, and prepares the OS environment. |
| [`k3s-master-setup`](k3s-master-setup)     | `masters`    | Installs, configures, and starts the K3s control plane on master nodes.                                                                                                 |
| [`k3s-fact-gathering`](k3s-fact-gathering) | `masters`    | Gathers runtime facts from the master node (such as the K3s node token and master IP) so worker nodes can use them to join the cluster.                                 |
| [`k3s-worker-setup`](k3s-worker-setup)     | `workers`    | Installs K3s agent and registers worker nodes to the cluster using the gathered master IP and token.                                                                    |
| [`k3s-node-labeling`](k3s-node-labeling)   | `masters`    | Post-join tasks to label and taint nodes dynamically based on inventory variables.                                                                                      |

---

## Core Cluster Services & GitOps

Services deployed on top of K3s to provide networking, certs, and deployment automation.

| Role                                   | Target Hosts | Description                                                                                                |
| -------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------- |
| [`argocd-setup`](argocd-setup)         | `masters[0]` | Installs ArgoCD for GitOps-driven deployment and synchronization of Kubernetes manifests and applications. |
| [`argocd-bootstrap`](argocd-bootstrap) | `masters[0]` | Bootstraps the K3s cluster by applying the ArgoCD root application (App of Apps) manifest.                 |
| [`openbao-setup`](openbao-setup)       | `masters[0]` | Installs OpenBao via Helm to provide open-source identity-based secret management, storage, and PKI.       |

---

## Developer Tooling & Registries

These roles deploy local developer tools and registries for hosting code and container images.

| Role                                           | Target Hosts | Description                                                                                   |
| ---------------------------------------------- | ------------ | --------------------------------------------------------------------------------------------- |
| [`k9s-setup`](k9s-setup)                       | `masters`    | Downloads and installs K9s terminal UI for Kubernetes cluster management.                     |
| [`local-k9s-setup`](local-k9s-setup)           | `localhost`  | Downloads and installs K9s terminal UI locally.                                               |
| [`openbao-secrets-init`](openbao-secrets-init) | `masters[0]` | Seeds initial secrets to OpenBao service.                                                     |
| [`ssh-config-setup`](ssh-config-setup)         | `localhost`  | Generates local `~/.ssh/config` and removes stale host key entries from `~/.ssh/known_hosts`. |
