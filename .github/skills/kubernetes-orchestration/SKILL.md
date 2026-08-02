---
name: "Kubernetes Orchestration"
description: "Managing K8s, namespaces, Helm, port-forwarding."
---
# K8s rules
- **Namespace**: Create explicitly using `kubernetes.core.k8s`.
- **Helm**: Set `create_namespace: false`.
- **Port-forward**: Run async (`async: 3600`, `poll: 0`). Verify via `async_status`. Wait for local port using `ansible.builtin.wait_for`.
