# k3s-node-labeling

An Ansible role that manages and reconciles K3s cluster node labels and taints based on inventory groups.

## Requirements

- `kubernetes.core` Ansible collection.
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

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: all
  roles:
    - role: k3s-node-labeling
```

## License

MIT
