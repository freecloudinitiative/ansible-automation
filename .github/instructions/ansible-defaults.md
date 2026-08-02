---
applyTo: roles/**/defaults/main.yml
---
# Ansible Defaults Best Practices

When defining default variables under `roles/**/defaults/main.yml`:

## 1. Naming Conventions
- **Variable Prefixing**: Prefix all variable names with the role name (e.g., `alloy_port_forward_enabled` in the `alloy-setup` role) to prevent namespace pollution and collision across roles.
- **Lowercase Names**: Use snake_case in lowercase for all variable keys.

## 2. Values & Documentation
- **Clear Defaults**: Provide sensible default values. If a value is optional or environment-dependent, set it to a safe fallback or use `omit`/null where appropriate.
- **Inline Comments**: Include short comments explaining the purpose, constraints, and valid types/formats for each variable.
