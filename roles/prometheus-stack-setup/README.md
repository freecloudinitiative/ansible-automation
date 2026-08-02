# prometheus-stack-setup

An Ansible role that deploys the `kube-prometheus-stack` Helm chart into a K3s cluster, including namespace creation, values templating, and Helm repo management.

## Features

- Creates the monitoring directory on the target host.
- Creates the `monitoring` Kubernetes namespace.
- Renders `kube-prometheus-values.yaml` from a Jinja2 template.
- Adds and updates the `prometheus-community` Helm repository.
- Installs `kube-prometheus-stack` via Helm into the `monitoring` namespace.

## Requirements

- Python 3.9+ on the target host.
- `kubernetes>=24.2.0`, `PyYAML>=3.11`, and `jsonpatch` Python packages.
- `kubernetes.core` Ansible collection.
- Helm installed on the target host (provided by the `k3s-master-setup` role).
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                           | Default Value                                        | Description                                     |
| ---------------------------------- | ---------------------------------------------------- | ----------------------------------------------- |
| `prometheus_stack_dir`             | `~/k3s-stack/monitoring`                             | Directory on the target host for stack files.   |
| `prometheus_namespace`             | `monitoring`                                         | Kubernetes namespace for the stack.             |
| `prometheus_release_name`          | `prometheus`                                         | Helm release name.                              |
| `prometheus_helm_repo_url`         | `https://prometheus-community.github.io/helm-charts` | Helm repository URL.                            |
| `prometheus_kubeconfig`            | `~/.kube/config`                                     | Path to the kubeconfig file on the target host. |
| `grafana_port_forward_enabled`     | `true`                                               | Whether to start background port-forwarding.    |
| `grafana_port_forward_address`     | `127.0.0.1`                                          | Listening IP address for port-forwarding.       |
| `grafana_port_forward_local_port`  | `3000`                                               | Local port to bind for port-forwarding.         |
| `grafana_port_forward_remote_port` | `80`                                                 | Remote port on the Grafana service.             |
| `grafana_service_name`             | `svc/kube-prometheus-stack-grafana`                  | Target service for port-forwarding.             |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: master
  become: true
  roles:
    - role: prometheus-stack-setup
```

## License

MIT
