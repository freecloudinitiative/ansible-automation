# k3s-node-labeling

An Ansible role that manages and reconciles K3s cluster node labels and taints based on inventory groups.

## Requirements

- `kubernetes.core` Ansible collection (pinned version recommended, e.g. `>= 2.4.0` or requirements file).
- Python dependencies installed on `groups['masters'][0]` (the control plane host performing delegated tasks):
  - Python 3.9+
  - `kubernetes` >= 24.2.0
  - `PyYAML` >= 3.11
  - `jsonpatch`
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                  | Default Value                                                         | Description                                                   |
| ------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------- |
| `kubeconfig_path`         | `/etc/rancher/k3s/k3s.yaml`                                           | Path to the kubeconfig file.                                  |
| `context`                 | `""`                                                                  | Kubernetes context to use.                                    |
| `node_labels.high_memory` | `{'node-tier': 'high-memory'}`                                        | Labels applied to nodes in the `high_memory` inventory group. |
| `node_labels.mid_memory`  | `{'node-tier': 'mid-memory'}`                                         | Labels applied to nodes in the `mid_memory` inventory group.  |
| `node_labels.low_memory`  | `{'node-tier': 'low-memory'}`                                         | Labels applied to nodes in the `low_memory` inventory group.  |
| `node_taints.low_memory`  | `[{'key': 'memory', 'value': 'limited', 'effect': 'NoSchedule'}]`     | Taints applied to `low_memory` nodes.                         |
| `node_taints.master`      | `[{'key': 'node-role.kubernetes.io/master', 'effect': 'NoSchedule'}]` | Taints applied to control plane / master nodes.               |
| `node_arch.masters`       | `arm64`                                                               | CPU architecture label for master nodes (`arm64` or `amd64`). |
| `node_arch.workers`       | `arm64`                                                               | CPU architecture label for worker nodes (`arm64` or `amd64`). |
| `node_arch.high_memory`   | `arm64`                                                               | CPU architecture label for high-memory nodes.                 |
| `node_arch.mid_memory`    | `arm64`                                                               | CPU architecture label for mid-memory nodes.                  |
| `node_arch.low_memory`    | `arm64`                                                               | CPU architecture label for low-memory nodes.                  |

## Dependencies

- `k3s-master-setup`

## Example Playbook

The inventory must define a `master` host group (with at least one control plane node, `groups['master'][0]`) for task delegation:

```ini
[master]
master1.example.com

[high_memory]
node1.example.com
```

```yaml
- hosts: master
  roles:
    - role: k3s-node-labeling
```

## License

MIT
