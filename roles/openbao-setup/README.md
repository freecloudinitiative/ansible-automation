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
| `port_forward_enabled` | `true` | Enable background port-forwarding |
| `port_forward_address` | `0.0.0.0` | Listening address for port-forwarding |
| `openbao_port_forward_local_port` | `8200` | Local port for port-forwarding |
| `openbao_port_forward_remote_port` | `8200` | Remote port for port-forwarding |

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
