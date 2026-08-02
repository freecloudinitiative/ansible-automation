---
applyTo: roles/*-setup/tasks/main.yml
---
# Kubernetes & Helm Setup Best Practices

For roles managing Kubernetes setups and Helm installations (e.g. `alloy-setup`, `argocd-setup`, `cert-manager-setup`, etc.):

## 1. Namespace Lifecycle
- **Explicit Creation**: Always provision the namespace explicitly using `kubernetes.core.k8s` before installing applications:
  ```yaml
  - name: Ensure namespace exists
    kubernetes.core.k8s:
      api_version: v1
      kind: Namespace
      name: "{{ target_namespace }}"
      state: present
  ```
- **Helm Namespace Flag**: When installing Helm charts via `kubernetes.core.helm`, set `create_namespace: false` to avoid conflicts with explicitly declared namespace manifests.

## 2. Service Exposure & Port Forwarding
- **Supervision & Validation**: When setting up local port-forwarding via `kubectl port-forward`:
  - Run it asynchronously (`async: 3600`, `poll: 0`).
  - Validate the job startup using `ansible.builtin.async_status`.
  - Verify the port accepts connections using `ansible.builtin.wait_for` on the local port before declaring success.
