# Ansible Automation

This repository hosts Ansible automations for the Free Cloud Initiative. It is structured to manage cluster nodes through comprehensive roles, playbooks, and inventory configurations.

## Key Components

- **Roles**: Modular and reusable Ansible roles for system configuration and deployment.
- **Playbooks**: High-level scripts defining automation workflows.
- **Inventory**: The `inventory.ini` file, which manages the information and groupings of cluster nodes.

## Usage

To execute a playbook that requires a vault password (for encrypted variables), use the `--ask-vault-pass` flag:

```bash
ansible-playbook <playbook_file_name> --ask-vault-pass
```
