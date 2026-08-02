# k3s-master-setup

Ansible role for initializing and configuring the primary master node in a High Availability (HA) K3s cluster.

## Features

- Downloads and runs the official K3s installation script (`https://get.k3s.io`).
- Initializes an embedded etcd HA cluster (`--cluster-init`).
- Disables default `metrics-server` and `traefik` to allow custom component deployments.
- Configures dynamic IP binding (`--node-ip`, `--advertise-address`, and `--tls-san`) using host Facts (`ansible_default_ipv4.address`).
- Enables etcd snapshot scheduling (`cron: 0 */6 * * *`, retention: 10).
- Configures secure kubeconfig permissions (`0640`) and API server RBAC authorization.
- Reads and registers the K3s node token (`/var/lib/rancher/k3s/server/node-token`) for worker/secondary master joins.

## Requirements

- Target OS: Linux (Ubuntu/Debian/RHEL/CentOS compatible).
- Root or `sudo` privileges on the target node (`become: true`).
- `/etc/k3s-bootstrap-token` pre-created or present if token file loading is required.

## Role Variables

| Variable                       | Default            | Description                                            |
| ------------------------------ | ------------------ | ------------------------------------------------------ |
| `ansible_default_ipv4.address` | Auto-gathered Fact | IP address used for cluster node configuration and SAN |

## Output Registered Variables / Facts

- `k3s_node_token`: Contains the slurped node token (`k3s_node_token.content | b64decode`) for joining additional nodes.
- `k3s_master_ip`: Fact variable storing the master node's IPv4 address (`{{ ansible_default_ipv4.address }}`).
