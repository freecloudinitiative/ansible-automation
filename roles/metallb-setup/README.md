# metallb-setup

An Ansible role that deploys MetalLB via Helm (`metallb/metallb`) into a K3s cluster and applies IP address pool and L2 advertisement configurations.

## Features

- Creates the load-balancer configuration directory on target host (`~/k3s-stack/load-balancer`).
- Adds and updates the `metallb` Helm repository (`https://metallb.github.io/metallb`).
- Renders `metallb-values.yaml` from Jinja2 template (`metallb-values.yaml.j2`).
- Renders `metallb-config.yaml` (`IPAddressPool` and `L2Advertisement`) from Jinja2 template (`metallb-config.yaml.j2`).
- Installs `metallb` via Helm into the `metallb-system` namespace.
- Applies the MetalLB custom resources (`IPAddressPool` & `L2Advertisement`).
- Optionally exposes MetalLB resources via background port-forwarding.

## Requirements

- Python 3.9+ on target host.
- Python packages on target host: `kubernetes>=24.2.0`, `PyYAML>=3.11`, `jsonpatch` (installed via `k3s-pre-setup` role).
- `kubernetes.core` Ansible collection.
- Helm installed on target host (provided by `k3s-master-setup`).
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                     | Default Value                                           | Description                                      |
| ---------------------------- | ------------------------------------------------------- | ------------------------------------------------ |
| `metallb_namespace`          | `metallb-system`                                        | Kubernetes namespace for MetalLB.                |
| `metallb_stack_dir`         | `~/k3s-stack/load-balancer`                             | Directory on target host for config files.       |
| `metallb_release_name`       | `metallb`                                               | Helm release name.                               |
| `metallb_helm_repo_url`      | `https://metallb.github.io/metallb`                    | Helm repository URL.                             |
| `kubeconfig_path`         | `{{ kubeconfig_path \| default(...) }}`                 | Path to the kubeconfig file on target host.      |
| `metallb_ip_pool_range`      | `192.168.1.100-192.168.1.110`                           | IP address range for `IPAddressPool`.            |
| `port_forward_enabled`| `true`                                                 | Whether to start background port-forwarding.    |
| `port_forward_address`| `0.0.0.0`                                             | Listening IP address for port-forwarding.       |
| `metallb_port_forward_local_port` | `80`                                               | Local port to bind for port-forwarding.         |
| `metallb_port_forward_remote_port`| `80`                                              | Remote port on target resource.                  |
| `metallb_service_name`        | `daemonset/speaker`                                     | Target resource for port-forwarding.             |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: masters
  become: true
  roles:
    - role: metallb-setup
```

## License

MIT
