# grafana-dashboards

Ansible role that imports Grafana dashboard JSON files into a running Grafana instance.

## Variables

| Variable                 | Default                                      | Description                                    |
| ------------------------ | -------------------------------------------- | ---------------------------------------------- |
| `grafana_url`            | `http://{{ k3s_master1_public_ip }}/grafana` | Grafana base URL                               |
| `grafana_admin_user`     | `admin`                                      | Grafana admin username                         |
| `grafana_admin_password` | `{{ grafana_admin_password }}`               | Grafana admin password (from vault/group_vars) |

## Requirements

Collection `community.grafana` must be installed:

```bash
ansible-galaxy collection install community.grafana
```
