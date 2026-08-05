# Ansible Role: gitea-setup

Installs Gitea Git service via Helm into a K3s cluster.

## Requirements

- K3s / Kubernetes cluster.
- `kubernetes.core` Ansible collection.
- `helm` and `kubectl` binaries on target hosts.

## Role Variables

| Variable                         | Default                        | Description                                    |
| -------------------------------- | ------------------------------ | ---------------------------------------------- |
| `gitea_namespace`                | `gitea`                        | Namespace to install Gitea                     |
| `gitea_release_name`             | `gitea`                        | Helm release name                              |
| `gitea_helm_repo_url`            | `https://dl.gitea.com/charts/` | Gitea Helm Repository URL                      |
| `kubeconfig_path`                | `/etc/rancher/k3s/k3s.yaml`    | Path to kubeconfig file                        |
| `gitea_node_tier`                | `mid-memory`                   | Target node tier for scheduling the Gitea pods |
| `gitea_actions_enabled`          | `true`                         | Enable Gitea Actions in app.ini config         |
| `port_forward_enabled`           | `true`                         | Enable backend background port-forwarding      |
| `port_forward_address`           | `0.0.0.0`                      | Local address to bind the port-forward         |
| `gitea_port_forward_local_port`  | `3001`                         | Local port to map Gitea port-forward           |
| `gitea_port_forward_remote_port` | `3000`                         | Remote container port to map                   |

## Dependencies

None.

## Example Playbook

```yaml
- name: Install Gitea
  hosts: masters
  become: true
  roles:
    - gitea-setup
```
