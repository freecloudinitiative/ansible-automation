# Ansible Role: argocd-bootstrap

Bootstraps the K3s cluster by applying the ArgoCD root application (App of Apps) manifest.

## Requirements

- `kubernetes.core` Ansible collection.
- An existing ArgoCD installation in the cluster (external prerequisite).

## Role Variables

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `argocd_bootstrap_enabled` | `true` | Enable or disable applying the root application. |
| `argocd_gitops_repo_url` | `https://github.com/freecloudinitiative/k3s-manifests.git` | URL to the GitOps manifest repository. |
| `argocd_gitops_repo_path` | `infrastructure` | Directory path containing core application/infrastructure manifests. |
| `argocd_gitops_repo_revision` | `HEAD` | Target revision (branch, tag, or commit hash) to track. |
| `argocd_namespace` | `argocd` | Target namespace for ArgoCD. |
| `kubeconfig_path` | `/etc/rancher/k3s/k3s.yaml` | Path to the kubeconfig file. |
| `context` | `""` | Kubernetes context to use. |

## Example Playbook

```yaml
- name: Bootstrap ArgoCD GitOps
  hosts: "{{ groups['masters'][0] }}"
  become: true
  roles:
    - argocd-bootstrap
```
