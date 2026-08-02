# loki-setup

An Ansible role that deploys the `grafana/loki` Helm chart into a K3s cluster, including configuration rendering and optional background port-forwarding.

## Features

- Creates the monitoring directory on the target host.
- Adds and updates the `grafana` Helm repository.
- Renders `loki-values.yaml` from a Jinja2 template (`loki-values.yaml.j2`).
- Installs `loki` via Helm into the `monitoring` namespace.
- Optionally exposes the `loki-gateway` service via background port-forwarding.

## Requirements

- Python 3.9+ on the target host.
- `kubernetes` Python package.
- `kubernetes.core` Ansible collection.
- Helm installed on the target host (provided by `k3s-master-setup`).
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                        | Default Value                           | Description                                     |
| ------------------------------- | --------------------------------------- | ----------------------------------------------- |
| `monitoring_namespace`          | `monitoring`                            | Kubernetes namespace for Loki installation.     |
| `loki_stack_dir`                | `~/k3s-stack/monitoring`                | Directory on target host for Loki config files. |
| `loki_release_name`             | `loki`                                  | Helm release name.                              |
| `loki_helm_repo_url`            | `https://grafana.github.io/helm-charts` | Helm repository URL.                            |
| `loki_kubeconfig`               | `{{ kubeconfig_path \| default(...) }}` | Path to the kubeconfig file on target host.     |
| `loki_port_forward_enabled`     | `true`                                  | Whether to start background port-forwarding.    |
| `loki_port_forward_address`     | `127.0.0.1`                             | Listening IP address for port-forwarding.       |
| `loki_port_forward_local_port`  | `3100`                                  | Local port to bind for port-forwarding.         |
| `loki_port_forward_remote_port` | `3100`                                  | Remote port on the Loki service.                |
| `loki_service_name`             | `svc/loki-gateway`                      | Target service for port-forwarding.             |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: masters
  become: true
  roles:
    - role: loki-setup
```

## License

MIT
