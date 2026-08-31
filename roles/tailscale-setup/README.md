# tailscale-setup

Installs Tailscale on every node (`hosts: all`) and joins it to the tailnet, so nodes are reachable over their Tailscale IP for SSH/`kubectl` without a public IP or port-forward. Per-node access only — no subnet routing, no exit node. Nothing about k3s or its network changes; this is purely an out-of-band admin access path.

## Usage

```bash
TAILSCALE_AUTH_KEY=tskey-auth-... ansible-playbook playbook.yml
```

Generate a **reusable**, **non-ephemeral** auth key at <https://login.tailscale.com/admin/settings/keys> — one key joins every node in the run; a single-use key would only work for the first host, and an ephemeral one drops nodes from the tailnet on reboot.

## What it does

1. Installs the Tailscale CLI (`tailscale.com/install.sh`) if not already present.
2. `tailscale up` with the auth key, skipped if the node is already connected.
3. Prints each node's Tailscale IP.

`--accept-dns=false` so Tailscale's MagicDNS doesn't override the node's existing resolver config. `--ssh` also turns on Tailscale SSH as a second access path, gated by the tailnet's own ACLs.
