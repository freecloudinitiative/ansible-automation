# Ansible Role: sealed-secret-setup

Installs Bitnami Sealed Secrets controller via Helm into a K3s cluster.

## Requirements

- K3s / Kubernetes cluster.
- `kubernetes.core` Ansible collection.
- `helm` and `kubectl` binaries on target hosts.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `sealed_secret_namespace` | `sealed-secrets` | Namespace to install Sealed Secrets controller |
| `sealed_secret_release_name` | `sealed-secrets` | Helm release name |
| `sealed_secret_helm_repo_url` | `https://bitnami.github.io/sealed-secrets` | Sealed Secrets Helm Repository URL |
| `sealed_secret_kubeconfig` | `/etc/rancher/k3s/k3s.yaml` | Path to kubeconfig file |
| `sealed_secret_node_tier` | `mid-memory` | Target node tier for scheduling the controller pod |
| `sealed_secret_controller_name` | `sealed-secrets` | Controller/Deployment resource name used for fullnameOverride |

## Dependencies

None.

## Example Playbook

```yaml
- name: Install Sealed Secrets
  hosts: masters
  become: true
  roles:
    - sealed-secret-setup
```
