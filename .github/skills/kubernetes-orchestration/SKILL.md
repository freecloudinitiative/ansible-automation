---
name: "Kubernetes Orchestration Skill"
description: "Triggered when configuring Kubernetes deployments, namespaces, Helm charts, or port-forwarding"
---
# Kubernetes & Helm Orchestration Guidelines

When this skill is active, apply the following workspace practices:

## 1. Declarative Management
- **Explicit Namespace Provisioning**: Always create namespaces explicitly using a `kubernetes.core.k8s` namespace task if a role installs components to a new namespace.
- **Helm Namespace Isolation**: Set `create_namespace: false` in `kubernetes.core.helm` module tasks to prevent namespace metadata overlaps.

## 2. Telemetry & Port-Forwarding Validation
- **Job Checking**: When starting a background `kubectl port-forward` command, run it asynchronously (`async: 3600`, `poll: 0`).
- **Validation Check**: Verify the port-forwarding job succeeds using `ansible.builtin.async_status` and wait for the local socket to accept connections using `ansible.builtin.wait_for` before finishing execution.
