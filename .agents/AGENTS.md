# Workspace Rules for Antigravity

This repository automates the provisioning of a Raspberry Pi-based K3s Kubernetes cluster and observability/security stack using Ansible.

## Target-Specific Guidelines

Detailed, file-pattern-matched guidelines are available under [.github/instructions/](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions):

- **Ansible Tasks**: [`ansible-tasks.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/ansible-tasks.md) (applies to `roles/**/tasks/main.yml`)
- **Kubernetes & Helm Setup**: [`kubernetes-helm.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/kubernetes-helm.md) (applies to `roles/*-setup/tasks/main.yml`)
- **K3s Clustering**: [`k3s-clustering.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/k3s-clustering.md) (applies to `roles/k3s-*/tasks/main.yml`)
- **Defaults & Variables**: [`ansible-defaults.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/ansible-defaults.md) (applies to `roles/**/defaults/main.yml`)
- **Role Metadata & Dependencies**: [`ansible-meta.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/ansible-meta.md) (applies to `roles/**/meta/main.yml`)

## Core Project Guidelines

### 1. Ansible & Playbook Best Practices
- **Host Targeting**: Ensure tasks run on correct target host groups (`masters`, `workers`, or `all`).
- **Privileged Tasks**: Always use `become: true` for OS configuration, package installations, and K3s control tasks.
- **Idempotency**: All tasks must be idempotent. Avoid shell commands where native Ansible modules exist.

### 2. Kubernetes & Helm Standards
- **Namespace Provisioning**: Always create namespaces explicitly using an `kubernetes.core.k8s` namespace task if a role installs components to a new namespace.
- **Parameter Unification**: Keep variables standardized across observability roles (e.g., namespace definitions, service names, and port-forwarding configs).
- **Helm Namespace Flag**: When installing Helm charts via the `kubernetes.core.helm` module, explicitly set `create_namespace: false`.
