# Ansible Role: argocd

An Ansible role that installs ArgoCD in a Kubernetes cluster using its official manifests, patches deployments to use a specific `nodeSelector`, and optionally sets up port-forwarding to access the ArgoCD UI.

## Requirements

- `kubernetes.core` Ansible collection must be installed.
- A valid kubeconfig file to connect to the Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                          | Default Value                                                                      | Description                                                          |
| --------------------------------- | ---------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `argocd_namespace`                | `argocd`                                                                           | The Kubernetes namespace where ArgoCD will be installed.             |
| `argocd_manifest_url`             | `https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` | URL to the official ArgoCD installation manifest.                    |
| `argocd_node_selector`            | `{'node-tier': 'mid-memory'}`                                                      | Node selector applied to the ArgoCD deployments.                     |
| `argocd_deployments`              | `['argocd-server', ...]`                                                           | List of ArgoCD deployments to patch with the `nodeSelector`.         |
| `kubeconfig_path`                 | `~/.kube/config`                                                                   | Path to the kubeconfig file.                                         |
| `context`                         | `""`                                                                               | The kubernetes context to use.                                       |
| `argocd_port_forward_enabled`     | `false`                                                                            | Whether to automatically set up port-forwarding for the ArgoCD UI.   |
| `argocd_port_forward_local_port`  | `8080`                                                                             | Local port used for port-forwarding to access the UI.                |
| `argocd_port_forward_remote_port` | `443`                                                                              | Remote service port (argocd-server) used for port-forwarding.        |

## Dependencies

- None

## Example Playbook

```yaml
- hosts: localhost
  roles:
    - role: argocd
```

## Outputs

When the installation is complete, the role will output the initial ArgoCD admin password required to log into the UI as the `admin` user. It will also provide the command needed to manually start port-forwarding if automatic port-forwarding (`argocd_port_forward_enabled`) was disabled.

## License

MIT
