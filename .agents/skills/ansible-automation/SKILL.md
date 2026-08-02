---
name: "Ansible Automation Skill"
description: "Triggered when editing, creating, or refactoring Ansible playbooks, roles, tasks, metadata, or defaults"
---
# Ansible Automation Guidelines

When this skill is active, apply the following workspace practices:

## 1. Task Development
- **Fully-Qualified Collection Names (FQCN)**: Write all module declarations with FQCN (e.g., `ansible.builtin.template`).
- **Idempotency**: Use specialized modules (such as `ansible.builtin.replace`, `ansible.builtin.lineinfile`, `ansible.builtin.file`) instead of raw shell execution.
- **Privileged Context**: Specify `become: true` for system configurations, service management, and software installations.

## 2. Parameterization
- **Defaults Prefixing**: Prefix role variables in `defaults/main.yml` with the role's name to prevent namespace collisions.
- **Metadata Dependencies**: Declare role dependencies explicitly in `meta/main.yml` using the standard YAML format.
