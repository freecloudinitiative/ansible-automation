# Ansible Role: harbor-setup

Installs Harbor container registry via Helm into a K3s cluster.

## Requirements

- K3s / Kubernetes cluster.
- `kubernetes.core` Ansible collection.
- `helm` and `kubectl` binaries on target hosts.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `harbor_namespace` | `harbor` | Namespace to install Harbor |
| `harbor_release_name` | `harbor` | Helm release name |
| `harbor_helm_repo_url` | `https://helm.goharbor.io` | Harbor Helm Repository URL |
| `harbor_version` | `v2.11.0` | Harbor component multi-arch image tag |
| `kubeconfig` | `/etc/rancher/k3s/k3s.yaml` | Path to kubeconfig file |
| `port_forward_enabled` | `true` | Enable background port-forwarding |
| `port_forward_address` | `0.0.0.0` | Listening address for port-forwarding |
| `harbor_node_tier` | `high-memory` | Target node tier for scheduling the Harbor pods |
| `harbor_expose_type` | `ingress` | Expose type (e.g. `ingress`, `clusterIP`, `nodePort`) |
| `harbor_external_url` | `http://harbor.local` | External URL to reach Harbor |
| `harbor_admin_password` | `Harbor12345` | Harbor Admin Password |

## Dependencies

None.

## Example Playbook

```yaml
- name: Install Harbor
  hosts: masters
  become: true
  roles:
    - harbor-setup
```
