# tempo-setup

An Ansible role that deploys the `grafana-community/tempo` Helm chart into a K3s cluster, including configuration rendering and optional background port-forwarding.

## Features

- Creates the monitoring directory on target host (`~/k3s-stack/monitoring`).
- Adds and updates the `grafana-community` Helm repository (`https://grafana-community.github.io/helm-charts`).
- Renders `tempo-values.yaml` from Jinja2 template (`tempo-values.yaml.j2`).
- Installs `tempo` via Helm into the `monitoring` namespace.
- Optionally exposes `tempo` service via background port-forwarding.

## Requirements

- Python 3.9+ on target host.
- `kubernetes` Python package.
- `kubernetes.core` Ansible collection.
- Helm installed on target host (provided by `k3s-master-setup`).
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                         | Default Value                                     | Description                                  |
| -------------------------------- | ------------------------------------------------- | -------------------------------------------- |
| `monitoring_namespace`           | `monitoring`                                      | Kubernetes namespace for Tempo installation. |
| `tempo_stack_dir`                | `~/k3s-stack/monitoring`                          | Directory on target host for config files.   |
| `tempo_release_name`             | `tempo`                                           | Helm release name.                           |
| `tempo_helm_repo_url`            | `https://grafana-community.github.io/helm-charts` | Helm repository URL.                         |
| `tempo_kubeconfig`               | `{{ kubeconfig_path \| default(...) }}`           | Path to the kubeconfig file on target host.  |
| `tempo_port_forward_enabled`     | `true`                                            | Whether to start background port-forwarding. |
| `tempo_port_forward_address`     | `127.0.0.1`                                       | Listening IP address for port-forwarding.    |
| `tempo_port_forward_local_port`  | `3200`                                            | Local port to bind for port-forwarding.      |
| `tempo_port_forward_remote_port` | `3200`                                            | Remote port on the target service.           |
| `tempo_service_name`             | `svc/tempo-gateway`                               | Target service for port-forwarding.          |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: masters
  become: true
  roles:
    - role: tempo-setup
```

## License

MIT
