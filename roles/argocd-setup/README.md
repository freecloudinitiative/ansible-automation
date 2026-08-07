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
| `port_forward_enabled`     | `true`                                                                             | Enable automatic background port-forwarding.      |
| `port_forward_address`     | `0.0.0.0`                                                                          | Listening IP address for port-forwarding.         |
| `argocd_port_forward_local_port`  | `8080`                                                                             | Local port for port-forwarding.                   |
| `argocd_port_forward_remote_port` | `443`                                                                              | Remote service port (argocd-server).              |
| `argocd_cli_version`              | `v2.10.4`                                                                          | ArgoCD CLI binary version to install.             |

## Dependencies

- `k3s-master-setup`

## Example Playbook

```yaml
- hosts: master
  roles:
    - role: argocd-setup
```

## Manual Port-Forward

If the ArgoCD port-forward process dies and you need to re-establish it manually, SSH into the master node and run:

```bash
# Kill any stale process
pkill -f "[k]ubectl port-forward.*8080:443" || true

# Start fresh port-forward in background
nohup kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443 > /tmp/argocd-pf.log 2>&1 &
```

Then access ArgoCD at `https://<master_public_ip>:8080`.

## License

MIT
