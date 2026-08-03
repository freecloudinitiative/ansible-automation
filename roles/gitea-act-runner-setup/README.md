# Ansible Role: gitea-act-runner-setup

Deploys and registers the Gitea Act Runner in Kubernetes to execute Gitea Actions workflows.

## Requirements

- K3s / Kubernetes cluster.
- Gitea server running with Actions enabled (`gitea-setup` role executed).
- `kubernetes.core` Ansible collection.
- `kubectl` binary on target hosts.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `gitea_act_runner_namespace` | `gitea` | Namespace to install Gitea Act Runner |
| `gitea_act_runner_name` | `gitea-act-runner` | Kubernetes Deployment name |
| `gitea_act_runner_kubeconfig` | `/etc/rancher/k3s/k3s.yaml` | Path to kubeconfig file |
| `gitea_act_runner_gitea_url` | `http://gitea-http.gitea.svc.cluster.local:3000` | Gitea instance HTTP URL |
| `gitea_act_runner_registration_token` | `""` | Optional explicit registration token |
| `gitea_act_runner_auto_generate_token` | `true` | Auto-generate token from Gitea pod if not provided |
| `gitea_act_runner_image` | `gitea/act_runner:latest` | Act Runner container image |
| `gitea_act_runner_node_tier` | `mid-memory` | Target node tier for scheduling the runner pod |
| `gitea_act_runner_labels` | `ubuntu-latest:docker://node:18-bullseye...` | Comma-separated runner labels |

## Dependencies

- `gitea-setup`

## Example Playbook

```yaml
- name: Install Gitea Act Runner
  hosts: masters[0]
  become: true
  roles:
    - gitea-setup
    - gitea-act-runner-setup
```
