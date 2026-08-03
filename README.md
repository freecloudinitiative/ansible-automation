# FreeCloud Initiative - Ansible Automation

This repository contains the Ansible automation for provisioning and managing a K3s cluster on Raspberry Pi nodes and cloud compute instances. It handles everything from initial node preparation and boot configuration to local SSH setup and core Kubernetes services (ArgoCD, Traefik, Cert-Manager, Prometheus/Grafana, Harbor, etc.).

## Repository Structure

| Path                                         | Type      | Description                                                                                         |
| :------------------------------------------- | :-------- | :-------------------------------------------------------------------------------------------------- |
| [**roles/**](./roles/)                       | Directory | Contains all individual Ansible roles for specific tasks and service installations.                 |
| [**collections/**](./collections/)           | Directory | Defines the required Ansible collections (`kubernetes.core`, `community.general`, `ansible.posix`). |
| [**group_vars/**](./group_vars/)             | Directory | Global and group-specific variable definitions (`group_vars/all/secret.yml`).                       |
| [**playbook.yml**](./playbook.yml)           | File      | The primary playbook for full K3s cluster deployment and service configuration.                     |
| [**ssh-config.yml**](./ssh-config.yml)       | File      | Playbook for generating local `~/.ssh/config` and clearing stale host keys from `known_hosts`.      |
| [**pi-boot.yml**](./pi-boot.yml)             | File      | Playbook for configuring Raspberry Pi-specific `config.txt` settings.                              |
| [**thermal-check.yml**](./thermal-check.yml) | File      | Playbook for checking Raspberry Pi CPU temperature and thermal status.                              |
| [**inventory.ini**](./inventory.ini)         | File      | Defines the cluster inventory, grouped by node roles (master/worker) and resources.                 |
| [**ansible.cfg**](./ansible.cfg)             | File      | Local Ansible configuration (inventory path, roles path, etc.).                                     |
| [**README.md**](./README.md)                 | File      | Main repository documentation.                                                                      |

---

## Vault & Secret Management

Sensitive variables, admin credentials (e.g. `harbor_admin_password`, `grafana_admin_password`), and cluster node IP mappings (`ssh_hosts`) are stored in `group_vars/all/secret.yml`.

### Encrypting Secrets

To encrypt the secrets file with Ansible Vault:

```bash
ansible-vault encrypt group_vars/all/secret.yml
```

To edit an encrypted secrets file:

```bash
ansible-vault edit group_vars/all/secret.yml
```

### Running Playbooks with Vault

```bash
ansible-playbook playbook.yml
```

---

## Local SSH Configuration Management

When cluster node IP addresses change, run the `ssh-config.yml` playbook on your local workstation to purge stale host keys from `~/.ssh/known_hosts` and regenerate `~/.ssh/config`:

```bash
ansible-playbook ssh-config.yml
```
