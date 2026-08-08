---
applyTo: roles/**/tasks/main.yml
---
# Tasks
- **FQCN**: Use fully qualified collection names.
- **Idempotency**: Use native modules. No shell/command unless necessary.
- **Change**: Define `changed_when`/`failed_when` for shell/command.
- **Privilege**: Use `become: true` for OS/pkg tasks.
- **Kubeconfig**: Provide explicit kubeconfig for kubectl/k8s modules.
