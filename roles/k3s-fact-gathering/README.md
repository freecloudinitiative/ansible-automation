# k3s-fact-gathering

Ansible role for retrieving the K3s node token and master IP address from the primary control plane host. This role allows worker node installation plays to dynamically obtain cluster join credentials even when executed independently of the master installation play.

## What it does

- Reads and slurps the K3s node token (`/var/lib/rancher/k3s/server/node-token`) from the control plane node.
- Registers facts for `k3s_master_ip` and `kubeconfig_path`.

## Requirements

- Target OS: Linux (Ubuntu/Debian compatible).
- Root or `sudo` privileges on the target master node (`become: true`).
- An operational K3s master node where `/var/lib/rancher/k3s/server/node-token` exists.

## Registered Facts

| Fact Name         | Description                                                                     |
| ----------------- | ------------------------------------------------------------------------------- |
| `k3s_node_token`  | slurped result containing the raw base64-encoded node token.                    |
| `k3s_master_ip`   | IPv4 address of the primary master node (`ansible_facts.default_ipv4.address`). |
| `kubeconfig_path` | Path to the kubeconfig file on the master node.                                 |

## License

MIT
