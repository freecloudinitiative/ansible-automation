# Ansible Roles Overview

This directory contains the Ansible roles used to provision and configure the K3s cluster.

## Roles summary

| Role                                                         | Description                                                                                                   |
| :----------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------ |
| **[raspberry-pi-boot-config](./raspberry-pi-boot-config)**   | Configures Raspberry Pi boot settings (`config.txt`) and handles necessary reboots.                           |
| **[k3s-presetup](./k3s-presetup)**                           | Performs initial system preparation like disabling swap and installing prerequisites before K3s installation. |
| **[k3s-bootstrap](./k3s-bootstrap)**                         | Generates and secures the initial K3s bootstrap token.                                                        |
| **[k3s-master-install](./k3s-master-install)**               | Installs and configures the primary K3s master node.                                                          |
| **[distribute-k3s-node-password](./distribute-k3s-node-password)** | Manages passwords or authentication credentials for K3s nodes.                                                |
| **[k3s-worker-setup](./k3s-worker-setup)**                   | Installs K3s agent on worker nodes and joins them to the K3s cluster.                                         |
| **[cleanup-bootstrap](./cleanup-bootstrap)**                   | Cleanup bootstrap. |
| **[cilium-setup](./cilium-setup)**                           | Installs Cilium CNI with WireGuard encryption on the K3s cluster.                                             |
| **[k3s-node-labeling](./k3s-node-labeling)**                 | Labels and taints K8s nodes for workload distribution and resource management.                                |
| **[sealed-secrets-setup](./sealed-secrets-setup)**           | Installs Bitnami Sealed Secrets controller in the K3s cluster and the kubeseal CLI tool.                      |
| **[argocd-setup](./argocd-setup)**                           | Installs ArgoCD from official manifests, patches deployments, and sets up optional port-forwarding.           |
| **[longhorn-setup](./longhorn-setup)**                       | Installs Longhorn block storage with node selections and configurable ingress via Helm.                       |
| **[traefik-setup](./traefik-setup)**                         | Deploys cert-manager and configures Traefik as the default IngressController via Helm.                        |
| **[kyverno-setup](./kyverno-setup)**                         | Installs Kyverno policy engine via Helm and verifies deployment and webhook readiness.                        |
