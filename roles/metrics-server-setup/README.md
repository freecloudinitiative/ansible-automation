# metrics-server-setup

Installs [metrics-server](https://github.com/kubernetes-sigs/metrics-server) via Helm onto K3s cluster.

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `metrics_server_helm_repo_url` | `"https://kubernetes-sigs.github.io/metrics-server/"` | Helm repository URL |
| `metrics_server_namespace` | `"kube-system"` | Namespace for metrics-server |
| `metrics_server_release_name` | `"metrics-server"` | Helm release name |
| `kubeconfig_path` | `"/etc/rancher/k3s/k3s.yaml"` | Path to kubeconfig |

## Dependencies

None.

## Example Playbook

```yaml
- hosts: masters[0]
  become: true
  roles:
    - metrics-server-setup
```
