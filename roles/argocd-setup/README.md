# argocd-setup

An Ansible role that installs ArgoCD in a Kubernetes cluster using official manifests, patches deployments with a custom `nodeSelector`, and retrieves initial admin credentials.

## Requirements

- `kubernetes.core` Ansible collection.
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                          | Default Value                                                                      | Description                                       |
| --------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------- |
| `argocd_namespace`                | `argocd`                                                                           | Kubernetes namespace where ArgoCD is installed.   |
| `argocd_manifest_url`             | `https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` | URL to official ArgoCD install manifest.          |
| `argocd_node_selector`            | `{'node-tier': 'mid-memory'}`                                                      | Node selector applied to ArgoCD deployments.      |
| `argocd_deployments`              | `['argocd-server', 'argocd-repo-server', ...]`                                     | List of deployments to patch with `nodeSelector`. |
| `kubeconfig_path`                 | `/etc/rancher/k3s/k3s.yaml`                                                        | Path to the kubeconfig file.                      |
| `context`                         | `""`                                                                               | Kubernetes context to use.                        |
| `argocd_port_forward_enabled`     | `false`                                                                            | Enable automatic background port-forwarding.      |
| `argocd_port_forward_local_port`  | `8080`                                                                             | Local port for port-forwarding.                   |
| `argocd_port_forward_remote_port` | `443`                                                                              | Remote service port (argocd-server).              |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: master
  roles:
    - role: argocd-setup
```

## License

MIT
