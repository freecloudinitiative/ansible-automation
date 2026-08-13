# argocd-setup

An Ansible role that installs ArgoCD in a Kubernetes cluster using official manifests, patches deployments with a custom `nodeSelector`, configures ConfigMap for path-based routing, and retrieves initial admin credentials.

## Requirements

- `kubernetes.core` Ansible collection.
- A functional K3s / Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                          | Default Value                                                                                  | Description                                       |
| --------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `argocd_version`                  | `v3.5.1`                                                                                       | ArgoCD version for manifest download and CLI.     |
| `argocd_namespace`                | `argocd`                                                                                       | Kubernetes namespace where ArgoCD is installed.   |
| `argocd_manifest_url`             | `https://raw.githubusercontent.com/argoproj/argo-cd/{{ argocd_version }}/manifests/install.yaml` | URL to official ArgoCD install manifest.          |
| `argocd_node_selector`            | `{'node-tier': 'high-memory'}`                                                                 | Node selector applied to ArgoCD deployments.      |
| `argocd_deployments`              | `['argocd-server', 'argocd-repo-server', ...]`                                                 | List of deployments to patch with `nodeSelector`. |
| `kubeconfig_path`                 | `/etc/rancher/k3s/k3s.yaml`                                                                    | Path to the kubeconfig file.                      |
| `context`                         | `""`                                                                                           | Kubernetes context to use.                        |
| `argocd_admin_password`           | `""`                                                                                           | Variable set dynamically with retrieved password. |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: master
  roles:
    - role: argocd-setup
```

## Useful Commands

To retrieve the initial ArgoCD admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## License

MIT
