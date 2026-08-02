---
applyTo: roles/**/tasks/main.yml
---
# Ansible Tasks Best Practices

When writing or editing Ansible tasks under `roles/**/tasks/main.yml`:

## 1. Syntax & Modules
- **Use FQCN (Fully Qualified Collection Names)**: Always specify full module names (e.g., `ansible.builtin.template` instead of `template`, `ansible.builtin.command` instead of `command`, `kubernetes.core.k8s` instead of `k8s`).
- **Idempotency**: Avoid raw shell commands. Use structured modules (e.g. `ansible.builtin.replace`, `ansible.builtin.file`, `ansible.builtin.apt`) wherever possible.
- **Changed Behavior**: When using `command` or `shell`, explicitly define `changed_when` and `failed_when` to keep runs clean and predictable.

## 2. Privilege & Context
- **Privileged Configuration**: Use `become: true` for OS level config, package management, service operations, and root-owned directories.
- **Kubeconfig Context**: When executing tasks interacting with Kubernetes (using `kubernetes.core.k8s*` or `kubectl`), ensure the environment `KUBECONFIG` variable is explicitly passed or configured via `kubeconfig` parameter.
