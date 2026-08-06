# k3s-master-setup

Ansible role for initializing the primary master node (`--cluster-init`) and joining secondary master nodes (`--server`) in a High Availability (HA) K3s cluster.

## Features

- Downloads and runs the official K3s installation script (`https://get.k3s.io`).
- Initializes an embedded etcd HA cluster on the primary master (`groups['masters'][0]`) using `--cluster-init`.
- Joins secondary master nodes (`groups['masters'][1:]`) to the HA cluster using `--server` and `--token`.
- Enables K3s Embedded Registry (Spegel) via `--embedded-registry` for peer-to-peer image mirroring across cluster nodes.
- Disables default `metrics-server` and `traefik` to allow custom component deployments.
- Configures dynamic IP binding (`--node-ip`, `--advertise-address`, and `--tls-san`) using host Facts (`ansible_facts.default_ipv4.address`).
- Sets dynamic `--node-name` using `inventory_hostname`.
- Configures secure kubeconfig permissions (`0644`) for `{{ kubeconfig_path }}` and non-root user ownership on `~/.kube/config`.
- Installs Helm on master nodes.

## Requirements

- Target OS: Linux (Ubuntu/Debian/RHEL/CentOS compatible).
- Root or `sudo` privileges on target nodes (`become: true`).

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `kubeconfig_path` | `/etc/rancher/k3s/k3s.yaml` | Path to the generated cluster kubeconfig file |
| `k3s_embedded_registry` | `true` | Enable K3s Embedded Registry (Spegel) peer-to-peer image mirroring |
| `k3s_master_ip` | Required for secondary masters | IP address of the primary master node |
| `k3s_node_token` | Required for secondary masters | K3s node token for joining the HA cluster |

## Output Registered Variables / Facts

- `k3s_node_token`: Contains the slurped node token (`k3s_node_token.content | b64decode`) for joining additional master/worker nodes.
- `k3s_master_ip`: Fact variable storing the primary master node's IPv4 address.
