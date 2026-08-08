# Ansible Role: openbao-setup

Installs OpenBao via Helm into a K3s cluster.

## Requirements

- K3s / Kubernetes cluster.
- `kubernetes.core` Ansible collection.
- `helm` and `kubectl` binaries on target master host.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `openbao_namespace` | `openbao` | Namespace to install OpenBao |
| `openbao_release_name` | `openbao` | Helm release name |
| `openbao_stack_dir` | `~/k3s-stack/openbao` | Directory to store rendered values file |
| `openbao_helm_repo_url` | `https://openbao.github.io/openbao-helm` | OpenBao Helm Repository URL |
| `openbao_chart_version` | `0.4.2` | OpenBao Helm Chart Version |
| `kubeconfig_path` | `/etc/rancher/k3s/k3s.yaml` | Path to kubeconfig file |
| `openbao_node_tier` | `mid-memory` | Target node tier for scheduling the OpenBao pod |
| `openbao_ui_enabled` | `true` | Enable OpenBao web UI |
| `openbao_node_port` | `30200` | NodePort to access OpenBao service |
| `openbao_cli_install` | `true` | Download and install OpenBao CLI binary |
| `openbao_cli_version` | `2.1.0` | OpenBao CLI release version |
| `openbao_cli_install_dir` | `/usr/local/bin` | Directory to install OpenBao CLI binary |

## Dependencies

None.

## Example Playbook

```yaml
- name: Install OpenBao
  hosts: masters[0]
  become: true
  roles:
    - openbao-setup
```
