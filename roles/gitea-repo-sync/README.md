# gitea-repo-sync

Ansible role to automatically import and mirror external Git repositories (HTTPS or SSH format) into Gitea using the Gitea REST API.

## Requirements

- Running Gitea instance accessible via HTTP/HTTPS.

## Features

- **HTTPS Standardization**: Automatically converts SSH clone URLs (`git@github.com:org/repo.git`) into HTTPS clone URLs (`https://github.com/org/repo.git`).
- **Idempotent**: Handles existing repositories gracefully (HTTP `409 Conflict` status code).
- **Flexible Input Formats**: Accepts raw string URLs (HTTPS/SSH) or structured dictionaries.

## Role Variables

| Variable               | Default                                           | Description                                         |
| ---------------------- | ------------------------------------------------- | --------------------------------------------------- |
| `gitea_url`            | `http://localhost:3001`                           | Gitea API Base URL                                  |
| `gitea_admin_user`     | `giteaadmin`                                      | Gitea admin username                                |
| `gitea_admin_password` | `{{ password \| default('GiteaAdminPass123!') }}` | Gitea admin password                                |
| `gitea_repos`          | `[]`                                              | List of repository URLs or dictionary items to sync |

### Example Repository List (`gitea_repos`)

```yaml
gitea_repos:
  # Simple HTTPS string
  - https://github.com/freecloudinitiative/k3s-manifests
  # Simple SSH string (automatically converted to HTTPS)
  - git@github.com:freecloudinitiative/ansible-automation.git
  # Full dictionary configuration
  - url: https://github.com/freecloudinitiative/k3s-manifests.git
    name: k3s-manifests-custom
    owner: giteaadmin
    private: false
    mirror: true
```

## Example Playbook

```yaml
- name: Sync Repositories to Gitea
  hosts: "{{ groups['masters'][0] }}"
  roles:
    - role: gitea-repo-sync
      vars:
        gitea_repos:
          - https://github.com/freecloudinitiative/k3s-manifests
          - git@github.com:freecloudinitiative/ansible-automation.git
```
