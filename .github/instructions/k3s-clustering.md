---
applyTo: roles/k3s-*/tasks/main.yml
---
# K3s Cluster Configuration Rules

For roles managing K3s installation, joining, and cluster configurations:

## 1. Master & Worker Orchestration
- **Token Sharing**: The master node writes the cluster join token, which is fetched by `k3s-fact-gathering` and passed dynamically to `k3s-worker-setup`. Do not hardcode the token.
- **Node IP configuration**: When starting the K3s server or agent command, explicitly pass `--node-ip={{ ansible_facts.default_ipv4.address }}` and `--node-name` using appropriate templates or host facts.

## 2. Idempotency & Labeling
- **Node Labeling**: When labeling/tainting nodes in `k3s-node-labeling`, ensure that the target node exists in the active nodes list (`cluster_nodes`) before executing the command to prevent errors.
- **Service State**: Avoid restarting K3s services unnecessarily unless configuration files have actually changed. Use Ansible handlers or check file states.
