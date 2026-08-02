# Rules
- **Task guidelines**: See [guidelines](../.github/instructions).
- **Targeting**: Run cluster-scoped plays on first master `groups['masters'][0]`.
- **Privilege**: Use `become: true` for OS configs, pkg installs, K3s control.
- **Idempotency**: Use native modules. No `shell`/`command` unless necessary.
- **K8s Namespaces**: Create explicitly via `kubernetes.core.k8s`. Set `create_namespace: false` in Helm.
- **Observability**: Unify variable names across monitoring roles.
