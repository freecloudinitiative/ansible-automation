# `ssh-config-setup` Role

Ansible role to generate and manage the local `~/.ssh/config` file and purge stale host keys from `~/.ssh/known_hosts` using `ssh-keygen -R <ip>` for workstations (macOS/Linux).

## Requirements

- Local workstation running macOS or Linux.
- OpenSSH client installed (`ssh-keygen`).

## Role Variables

Available variables and their default values (see `defaults/main.yml`):

| Variable | Default Value | Description |
|---|---|---|
| `ssh_config_default_user` | `"entelektuelmaganda"` | Default SSH user for host entries |
| `ssh_config_default_identity_file` | `"~/.ssh/gcp_k3s"` | Default identity key file path |
| `ssh_config_dir` | `"{{ '~/.ssh' \| expanduser }}"` | Path to local SSH configuration directory |
| `ssh_config_file` | `"{{ '~/.ssh/config' \| expanduser }}"` | Path to local `config` file |
| `ssh_known_hosts_file` | `"{{ '~/.ssh/known_hosts' \| expanduser }}"` | Path to local `known_hosts` file |
| `ssh_config_clean_known_hosts` | `false` | Purges old keys for target host IPs using `ssh-keygen -R` |
| `ssh_config_scan_known_hosts` | `true` | Auto-scans host keys using `ssh-keyscan` to pre-trust target servers |
| `ssh_config_strict_host_key_checking` | `"no"` | Configures `StrictHostKeyChecking` in SSH config so first connections never prompt |
| `ssh_hosts` | `{}` | Map of hostname to IP address (supplied via playbook `vars` or `group_vars`). Entries with an empty IP (`""`) are skipped. |
| `ssh_config_custom_hosts` | `[]` | Additional custom SSH host blocks (e.g. `github.com`) |

## Example Usage

### Playbook (`ssh-config.yml`)

```yaml
---
- name: Configure Local SSH & Clean Known Hosts
  hosts: localhost
  connection: local
  become: false
  vars:
    ssh_config_clean_known_hosts: true
    ssh_hosts:
      master: "34.72.134.198"
      worker-1: "35.188.150.0"
      worker-2: "34.136.163.177"
      worker-3: "35.223.77.196"
  roles:
    - ssh-config-setup
```

### Overriding Host IP Mapping

```yaml
- name: Configure Local SSH
  hosts: localhost
  connection: local
  become: false
  vars:
    ssh_hosts:
      master: "34.72.134.198"
      master-2: "34.72.135.10"
      worker-1: "35.188.150.0"
      worker-2: "34.136.163.177"
      worker-3: "35.223.77.196"
      worker-test: "34.66.180.82"
  roles:
    - ssh-config-setup
```
