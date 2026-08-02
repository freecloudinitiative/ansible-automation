# Ansible Role: traefik-setup

Installs Traefik Ingress Controller via Helm into a K3s cluster.

## Requirements

- `kubernetes.core` Ansible collection.
- A functional K3s / Kubernetes cluster.

## Role Variables

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `traefik_namespace` | `traefik` | Kubernetes namespace for Traefik |
| `traefik_release_name` | `traefik` | Helm release name |
| `traefik_stack_dir` | `~/k3s-stack/traefik` | Stack directory for generated values |
| `traefik_helm_repo_url` | `https://traefik.github.io/charts` | Traefik Helm Repository URL |
| `traefik_chart_version` | `34.0.0` | Traefik Helm chart version |
| `traefik_image_tag` | `v3.3.2` | Traefik container image tag |
| `traefik_kubeconfig` | `/etc/rancher/k3s/k3s.yaml` | Path to the kubeconfig file |
| `traefik_node_tier` | `mid-memory` | Target node tier for scheduling Traefik |
| `traefik_dashboard_enabled` | `true` | Enable or disable Traefik dashboard ingressRoute |
| `traefik_ingress_class_enabled` | `true` | Enable IngressClass |
| `traefik_ingress_class_is_default` | `true` | Set Traefik as the default IngressClass |

## Dependencies

None.

## Example Playbook

```yaml
- name: Install Traefik Ingress Controller
  hosts: "{{ groups['masters'][0] }}"
  become: true
  roles:
    - traefik-setup
```
