# Ansible Role: longhorn-setup

An Ansible role that installs Longhorn distributed block storage in a Kubernetes cluster using the official Helm chart. This role allows configuring disk exclusions for designated nodes and provides options for replica counts, backup targets, and Ingress setup. 

## Requirements

- `kubernetes.core` Ansible collection must be installed.
- Helm must be installed and initialized on the control node.
- A valid kubeconfig file to connect to your Kubernetes cluster.

## Role Variables

Variables available for configuration, along with their default values (see `defaults/main.yml`):

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `longhorn_namespace` | `longhorn-system` | The Kubernetes namespace where Longhorn will be installed. |
| `longhorn_exclude_nodes` | `[]` | List of nodes (by hostname) to strictly exclude from Longhorn's default disk creation using the `node.longhorn.io/create-default-disk: "false"` label. |
| `longhorn_replica_count` | `2` | Default replica count for Longhorn volumes. |
| `longhorn_backup_target` | `""` | The URL/path for the Longhorn backup target (e.g., S3 bucket or NFS URL). |
| `longhorn_backup_secret` | `""` | Name of the Kubernetes Secret containing AWS/S3 backup credentials. |
| `longhorn_ingress_enabled` | `false` | Whether to enable an Ingress route for the Longhorn UI. |
| `longhorn_ingress_class` | `"nginx"` | Ingress class name to use if `longhorn_ingress_enabled` is true. |
| `longhorn_ingress_host` | `"longhorn.example.com"` | The hostname/FQDN for Longhorn UI Ingress. |
| `longhorn_wait_condition` | `"condition=available"`| The wait condition for checking if Longhorn has installed successfully via Helm. |
| `longhorn_timeout` | `"10m"` | Total timeout waiting for Longhorn resources to be ready. |
| `kubeconfig_path` | `~/.kube/config` | Path to the kubeconfig file used. |
| `context` | `""` | Specific Kubernetes context to use (optional). |

## Dependencies

- None.

## Example Playbook

```yaml
- hosts: localhost
  roles:
    - role: longhorn-setup
      vars:
        longhorn_exclude_nodes: 
          - master1.local
          - utility-node.local
        longhorn_replica_count: 3
        longhorn_ingress_enabled: true
        longhorn_ingress_host: "storage.mycluster.domain"
```

## Outputs

When the installation successfully completes, the role outputs the readiness status of the `longhorn-manager` and `longhorn-ui` deployments, along with the command needed to access the Longhorn User Interface (via Ingress URL if enabled, otherwise through `kubectl port-forward`).

## License

MIT
