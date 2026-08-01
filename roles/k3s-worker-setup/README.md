# k3s-worker-setup

Ansible role for joining and configuring worker agent nodes to an existing K3s cluster.

## Features

- Downloads and runs the official K3s installation script (`https://get.k3s.io`).
- Configures node name dynamically via `worker_label` (e.g., `worker-1`, `worker-2`).
- Binds node IP dynamically using host IPv4 facts (`ansible_default_ipv4.address`).
- Connects agent nodes to the master node API using `K3S_URL` and `K3S_TOKEN`.

## Requirements

- Target OS: Linux (Ubuntu/Debian/Raspbian compatible).
- Root or `sudo` privileges on target worker nodes (`become: true`).
- Pre-existing K3s master node providing `k3s_master_ip` and `k3s_node_token`.

## Required Variables

| Variable | Description |
| --- | --- |
| `worker_label` | Unique node name for the Kubernetes cluster defined in host inventory (e.g. `worker-1`) |
| `k3s_master_ip` | IP address of the primary K3s master server |
| `k3s_node_token` | Token retrieved from the primary K3s master server (`/var/lib/rancher/k3s/server/node-token`) |
| `ansible_default_ipv4.address` | Auto-gathered host IP address passed to `--node-ip` |
