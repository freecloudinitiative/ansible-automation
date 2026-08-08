---
name: "Ansible Automation"
description: "Editing playbooks, roles, tasks, defaults, meta."
---

# Ansible rules

- **FQCN**: Use fully qualified collection names (e.g. `ansible.builtin.template`).
- **Idempotency**: Use modules. Avoid `command`/`shell` unless necessary.
- **Privilege**: Use `become: true` for OS/service tasks.
- **Meta**: Declare role dependencies in `meta/main.yml`.
