# k3s-worker-setup

Ansible role for joining and configuring worker agent nodes to an existing K3s cluster.

## Features

- Downloads and runs the official K3s installation script (`https://get.k3s.io`).
- Configures node name dynamically via `worker_label` (e.g., `worker-1`, `worker-2`).
- Binds node IP dynamically using host IPv4 facts (`ansible_default_ipv4.address`).
- Connects agent nodes to the master node API using `K3S_URL` and `K3S_TOKEN`.
- Enables K3s Embedded Registry (Spegel) via `--embedded-registry` for peer-to-peer image mirroring.
- Waits for the master's `:6443` to accept connections, raises `k3s-agent`'s
  `TimeoutStartSec` to 300s via a systemd drop-in, and joins workers one at a
  time (`throttle: 1`) — on Raspberry Pi hardware the TLS bootstrap plus
  first-boot image pulls can outrun systemd's default 90s notify timeout when
  every worker joins a fresh master at once, which otherwise fails the first
  `ansible-playbook` run and only succeeds on a rerun. If the installer's own
  `systemctl start` still fails, the role restarts `k3s-agent` and waits up
  to four minutes for it to report active before failing loudly.

## Requirements

- Target OS: Linux (Ubuntu/Debian/Raspbian compatible).
- Root or `sudo` privileges on target worker nodes (`become: true`).
- Pre-existing K3s master node providing `k3s_master_ip` and `k3s_node_token`.

## Required Variables

| Variable | Default | Description |
| --- | --- | --- |
| `worker_label` | Required | Unique node name for the Kubernetes cluster defined in host inventory (e.g. `worker-1`) |
| `k3s_master_ip` | Required | IP address of the primary K3s master server |
| `k3s_node_token` | Required | Token retrieved from the primary K3s master server (`/var/lib/rancher/k3s/server/node-token`) |
| `k3s_embedded_registry` | `true` | Enable K3s Embedded Registry (Spegel) peer-to-peer image mirroring |
| `ansible_default_ipv4.address` | Auto-gathered | Host IP address passed to `--node-ip` |
