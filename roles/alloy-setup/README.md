# alloy-setup

An Ansible role that deploys the `grafana/alloy` Helm chart into a K3s cluster, including configuration rendering and optional background port-forwarding.

## Features

- Creates the monitoring directory on target host (`~/k3s-stack/monitoring`).
- Adds and updates the `grafana` Helm repository.
- Renders `alloy-values.yaml` from Jinja2 template (`alloy-values.yaml.j2`).
- Installs `alloy` via Helm into the `monitoring` namespace.
- Optionally exposes `alloy` via background port-forwarding.

## Requirements

- Python 3.9+ on target host.
- `kubernetes` Python package.
- `kubernetes.core` Ansible collection.
- Helm installed on target host (provided by `k3s-master-setup`).
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                         | Default Value                           | Description                                  |
| -------------------------------- | --------------------------------------- | -------------------------------------------- |
| `monitoring_namespace`           | `monitoring`                            | Kubernetes namespace for Alloy installation. |
| `alloy_stack_dir`                | `~/k3s-stack/monitoring`                | Directory on target host for config files.   |
| `alloy_release_name`             | `alloy`                                 | Helm release name.                           |
| `alloy_helm_repo_url`            | `https://grafana.github.io/helm-charts` | Helm repository URL.                         |
| `alloy_kubeconfig`               | `{{ kubeconfig_path \| default(...) }}` | Path to the kubeconfig file on target host.  |
| `alloy_port_forward_enabled`     | `true`                                  | Whether to start background port-forwarding. |
| `alloy_port_forward_address`     | `127.0.0.1`                             | Listening IP address for port-forwarding.    |
| `alloy_port_forward_local_port`  | `12345`                                 | Local port to bind for port-forwarding.      |
| `alloy_port_forward_remote_port` | `12345`                                 | Remote port on the target service/resource.  |
| `alloy_service_name`             | `daemonset/alloy`                       | Target resource for port-forwarding.         |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: masters
  become: true
  roles:
    - role: alloy-setup
```

## License

MIT
