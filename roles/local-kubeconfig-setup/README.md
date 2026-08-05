# local-kubeconfig-setup

Fetches the cluster `kubeconfig` from the primary K3s master node, replaces the loopback IP (`127.0.0.1`) with the public IP address of the master, saves it to `~/.kube/config-freecloud` on your local machine, and installs `k9s` via Homebrew on macOS if missing.

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `kubeconfig_remote_path` | `"/etc/rancher/k3s/k3s.yaml"` | Path to kubeconfig on the remote master |
| `local_kubeconfig_path` | `"~/.kube/config-freecloud"` | Target file path on local controller |
| `install_local_k9s` | `true` | Attempt installing k9s locally via Homebrew |
| `k9s_namespaces` | `[gitea, argocd, kube-system, monitoring, traefik]` | Default namespaces for K9s view |
| `k9s_config` | `{ ... }` | K9s options (refreshRate, thresholds, UI flags, etc.) |

## Example Playbook

```yaml
- hosts: "{{ groups['masters'][0] }}"
  become: true
  roles:
    - local-kubeconfig-setup
```

