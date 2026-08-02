# GitHub Copilot Instructions

This repository automates the provisioning of a Raspberry Pi-based K3s Kubernetes cluster and observability/security stack using Ansible.

## Structured Guidelines

Specific, pattern-matched guidelines are separated under [.github/instructions/](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions):

1. **Ansible Tasks & Styling**: See [`ansible-tasks.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/ansible-tasks.md) (targets `roles/**/tasks/main.yml`)
2. **Kubernetes & Helm Setup**: See [`kubernetes-helm.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/kubernetes-helm.md) (targets `roles/*-setup/tasks/main.yml`)
3. **K3s Clustering & Node Configuration**: See [`k3s-clustering.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/k3s-clustering.md) (targets `roles/k3s-*/tasks/main.yml`)
4. **Defaults & Variable Definitions**: See [`ansible-defaults.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/ansible-defaults.md) (targets `roles/**/defaults/main.yml`)
5. **Role Metadata & Dependencies**: See [`ansible-meta.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/instructions/ansible-meta.md) (targets `roles/**/meta/main.yml`)

## Custom Skills

Dynamic, context-based skills are configured under [.github/skills/](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/skills):
- **Ansible Automation Skill**: [`ansible-automation/SKILL.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/skills/ansible-automation/SKILL.md)
- **Kubernetes Orchestration Skill**: [`kubernetes-orchestration/SKILL.md`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/.github/skills/kubernetes-orchestration/SKILL.md)


## Core Architectural Rules

- **Host Target Safety**: Never run Kubernetes manifests directly on worker nodes; deploy stack manifests from the master node.
- **Resource Limitations**: Raspberry Pi nodes have limited resources. Avoid heavy configurations or bloated deployments.
- **FQCN Standard**: Always use Fully Qualified Collection Names for Ansible tasks.
