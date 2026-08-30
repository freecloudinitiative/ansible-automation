# `ssh-config-setup` Role

Ansible role to generate and manage the local `~/.ssh/config` file and purge stale host keys from `~/.ssh/known_hosts` using `ssh-keygen -R <ip>` for workstations (macOS/Linux).

Picks SSH `User` and `IdentityFile` from `ssh_config_cloud` (`gcp` or `aws`). A host may set `cloud:` to override the global value.

## Requirements

- Local workstation running macOS or Linux.
- OpenSSH client installed (`ssh-keygen`).

## Role Variables

Available variables and their default values (see `defaults/main.yml`):

| Variable | Default Value | Description |
|---|---|---|
| `ssh_config_cloud` | `"gcp"` | Global cloud. `gcp` or `aws`. Set in `group_vars/all/main.yml`. |
| `ssh_config_gcp_user` | `"entelektuelmaganda"` | SSH user when cloud is `gcp`. From vault. |
| `ssh_config_gcp_identity_file` | `"~/.ssh/gcp_k3s"` | SSH key when cloud is `gcp`. From vault. |
| `ssh_config_aws_user` | `"ubuntu"` | SSH user when cloud is `aws`. From vault. |
| `ssh_config_aws_identity_file` | `"~/.ssh/ansible_keypair.pem"` | SSH key when cloud is `aws`. From vault. |
| `ssh_config_identities_only` | `"yes"` | `IdentitiesOnly` on generated host blocks |
| `ssh_config_dir` | `"{{ '~/.ssh' \| expanduser }}"` | Path to local SSH configuration directory |
| `ssh_config_file` | `"{{ '~/.ssh/config' \| expanduser }}"` | Path to local `config` file |
| `ssh_known_hosts_file` | `"{{ '~/.ssh/known_hosts' \| expanduser }}"` | Path to local `known_hosts` file |
| `ssh_config_clean_known_hosts` | `false` | Purges old keys for target host IPs using `ssh-keygen -R` |
| `ssh_config_scan_known_hosts` | `true` | Auto-scans host keys using `ssh-keyscan` to pre-trust target servers |
| `ssh_config_strict_host_key_checking` | `"no"` | Configures `StrictHostKeyChecking` in SSH config so first connections never prompt |
| `ssh_hosts` | `{}` | Map of hostname to IP string or `{hostname, cloud, user, identity_file}`. Empty IP (`""`) skipped. |
| `ssh_config_custom_hosts` | `[]` | Additional custom SSH host blocks (e.g. `github.com`) |

## Vault: read or change SSH user and key path

`group_vars/all/secret.yml` is Ansible Vault. Playbooks prompt because `ansible.cfg` has `ask_vault_pass = true`.

View (does not write):

```bash
ansible-vault view group_vars/all/secret.yml
```

Edit and re-encrypt on save:

```bash
ansible-vault edit group_vars/all/secret.yml
```

Print only the resolved SSH vars Ansible will use:

```bash
ansible localhost -m ansible.builtin.debug -a 'var=ssh_config_cloud' --ask-vault-pass
ansible localhost -m ansible.builtin.debug -a 'var=ssh_config_aws_user' --ask-vault-pass
ansible localhost -m ansible.builtin.debug -a 'var=ssh_config_aws_identity_file' --ask-vault-pass
ansible localhost -m ansible.builtin.debug -a 'var=ssh_config_gcp_user' --ask-vault-pass
ansible localhost -m ansible.builtin.debug -a 'var=ssh_config_gcp_identity_file' --ask-vault-pass
```

If the file is plaintext for review, encrypt before commit:

```bash
ansible-vault encrypt group_vars/all/secret.yml
```

The pre-commit hook refuses an unencrypted `secret.yml`.

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
      master-1:
        hostname: "34.72.134.198"
        cloud: aws
      worker-1:
        hostname: "35.188.150.0"
        cloud: aws
      worker-2:
        hostname: "34.136.163.177"
        cloud: gcp
  roles:
    - ssh-config-setup
```

Per-host `cloud` wins over `ssh_config_cloud`. Per-host `user` / `identity_file` win over the vault values for that cloud.
