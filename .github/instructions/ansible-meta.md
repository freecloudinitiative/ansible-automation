---
applyTo: roles/**/meta/main.yml
---
# Ansible Meta Best Practices

When managing metadata and dependencies under `roles/**/meta/main.yml`:

## 1. Role Dependencies
- **Explicit Dependencies**: List other roles this role depends on inside the `dependencies` block. For example, if a role requires cluster setup, define it:
  ```yaml
  dependencies:
    - role: k3s-pre-setup
  ```
- **Avoid Duplication**: Ensure that dependencies are not redundant or causing infinite loops across roles.

## 2. Galaxy Metadata
- **Galaxy Fields**: Populate standard fields in `galaxy_info` such as `author`, `description`, `company`, `license`, and `min_ansible_version`.
- **Platforms**: Specify supported platforms (e.g. `Debian` or `Ubuntu`) and tags to make the roles searchable and organized.
