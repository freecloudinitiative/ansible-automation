---
applyTo: roles/*-setup/tasks/main.yml
---
# K8s & Helm
- **Namespace**: Create explicitly using `kubernetes.core.k8s`.
- **Helm**: Set `create_namespace: false`.
- **Port-forward**: Run async (`async: 3600`, `poll: 0`). Verify status via `async_status`. Wait for local port using `ansible.builtin.wait_for`.
