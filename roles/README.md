# Ansible Roles Overview

This directory contains the Ansible roles used to provision and configure the K3s cluster.

## Roles summary

| Role                                                         | Description                                                                                                   |
| :----------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------ |
| **[raspberry-pi-boot-config](./raspberry-pi-boot-config)**   | Configures Raspberry Pi boot settings (`config.txt`) and handles necessary reboots.                           |
| **[k3s-presetup](./k3s-presetup)**                           | Performs initial system preparation like disabling swap and installing prerequisites before K3s installation. |
| **[k3s-bootstrap](./k3s-bootstrap)**                         | Generates and secures the initial K3s bootstrap token.                                                        |
| **[k3s-master-install](./k3s-master-install)**               | Installs and configures the primary K3s master node.                                                          |
| **[distribute-node-passwords](./distribute-node-passwords)** | Manages passwords or authentication credentials for K3s nodes.                                                |
| **[k3s-worker-setup](./k3s-worker-setup)**                   | Installs K3s agent on worker nodes and joins them to the K3s cluster.                                         |
| **[cilium-setup](./cilium-setup)**                           | Installs Cilium CNI with WireGuard encryption on the K3s cluster.                                             |
| **[k3s-node-labeling](./k3s-node-labeling)**                 | Labels and taints K8s nodes for workload distribution and resource management.                                |
