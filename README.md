# FreeCloud Initiative - Ansible Automation

This repository contains the Ansible automation for provisioning and managing a K3s cluster on Raspberry Pi nodes. It handles everything from initial node preparation and boot configuration to the installation of core Kubernetes services like storage (Longhorn), networking (Cilium), and GitOps (ArgoCD).

## Repository Structure

| Path                                     | Type      | Description                                                                                         |
| :--------------------------------------- | :-------- | :-------------------------------------------------------------------------------------------------- |
| [**roles/**](./roles/)                   | Directory | Contains all individual Ansible roles for specific tasks and service installations.                 |
| [**collections/**](./collections/)       | Directory | Defines the required Ansible collections (`kubernetes.core`, `community.general`, `ansible.posix`). |
| [**group_vars/**](./group_vars/)         | Directory | Global and group-specific variable definitions.                                                     |
| [**playbook.yml**](./playbook.yml)       | File      | The primary playbook for full K3s cluster deployment and service configuration.                     |
| [**boot_config.yml**](./boot_config.yml) | File      | Playbook for configuring Raspberry Pi specific `config.txt` settings.                               |
| [**inventory.ini**](./inventory.ini)     | File      | Defines the cluster inventory, grouped by node roles (master/worker) and resources.                 |
| [**ansible.cfg**](./ansible.cfg)         | File      | Local Ansible configuration (inventory path, roles path, etc.).                                     |
| [**README.md**](./README.md)             | File      | This file.                                                                                          |
