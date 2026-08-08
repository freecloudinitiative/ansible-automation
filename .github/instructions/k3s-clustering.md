---
applyTo: roles/k3s-*/tasks/main.yml
---
# K3s Clustering
- **Token**: Share master token dynamically via facts. No hardcoding.
- **Node IP**: Pass `--node-ip={{ ansible_facts.default_ipv4.address }}` and `--node-name`.
- **Labels**: Verify node exists in `cluster_nodes` before labeling/tainting.
- **State**: Only restart services on config change (use handlers).
