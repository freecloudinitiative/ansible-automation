# Ansible Role: argocd-bootstrap

Bootstraps the K3s cluster by applying the ArgoCD root application (App of Apps) manifest.

## Requirements

- `kubernetes.core` Ansible collection.
- A functional K3s / Kubernetes cluster with ArgoCD installed.

## Role Variables

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `argocd_bootstrap_enabled` | `true` | Enable or disable applying the root application. |
| `argocd_root_app_manifest_url` | `https://raw.githubusercontent.com/freecloudinitiative/k3s-manifests/main/bootstrap/root-app.yaml` | Raw URL to the root application manifest. |
| `argocd_namespace` | `argocd` | Target namespace for ArgoCD. |
| `kubeconfig_path` | `/etc/rancher/k3s/k3s.yaml` | Path to the kubeconfig file. |
| `context` | `""` | Kubernetes context to use. |

## Dependencies

- `argocd-setup`

## Example Playbook

```yaml
- name: Bootstrap ArgoCD GitOps
  hosts: masters
  become: true
  roles:
    - argocd-bootstrap
```
