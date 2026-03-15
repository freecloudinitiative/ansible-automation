# Ansible Role: traefik-setup

An Ansible role that installs `cert-manager` using its official manifest and deploys `Traefik` as the default Ingress controller using its Helm chart. It allows applying specific node selectors and automatically waits for the deployments to be ready.

## Requirements

- `kubernetes.core` Ansible collection must be installed.
- Helm must be installed and initialized on the control node.
- A valid kubeconfig file to connect to the Kubernetes cluster.

## Role Variables

Variables available for configuration, along with their default values (see `defaults/main.yml`):

| Variable                        | Default Value                                                                             | Description                                                              |
| ------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `cert_manager_manifest_url`     | `https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml` | URL for the `cert-manager` deployment manifest.                          |
| `traefik_namespace`             | `traefik`                                                                                 | The Kubernetes namespace where Traefik will be installed.                |
| `traefik_repo_name`             | `traefik`                                                                                 | Name of the Traefik Helm repository.                                     |
| `traefik_repo_url`              | `https://helm.traefik.io/traefik`                                                         | URL of the Traefik Helm repository.                                      |
| `traefik_release_name`          | `traefik`                                                                                 | Helm release name for Traefik.                                           |
| `traefik_chart`                 | `traefik/traefik`                                                                         | Helm chart reference for Traefik.                                        |
| `traefik_node_selector`         | `{'node-tier': 'mid-memory'}`                                                             | Node selector applied to the Traefik deployment.                         |
| `traefik_ingress_class_default` | `true`                                                                                    | Whether to set this Traefik instance as the default IngressController.   |
| `traefik_wait_condition`        | `{'type': 'Available', 'status': 'true'}`                                                 | The condition used to determine if the Traefik installation is complete. |
| `traefik_timeout`               | `5m`                                                                                      | Timeout for Traefik Helm wait condition.                                 |
| `kubeconfig_path`               | `~/.kube/config`                                                                          | Path to the kubeconfig file used.                                        |
| `context`                       | `""`                                                                                      | Specific Kubernetes context to use (optional).                           |

```

## Outputs

When the installation is successfully completed, the role outputs a summary indicating `cert-manager` deployment status, `Traefik` deployment namespace, and confirms whether the `IngressClass` is set as default.
```
