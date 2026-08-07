# Ansible Role: openbao-secrets-init

Seeds initial secrets into OpenBao secret storage via REST API.

## Requirements

- Running OpenBao instance accessible at `openbao_url`.

## Role Variables

| Variable          | Default                        | Description                                                   |
| ----------------- | ------------------------------ | ------------------------------------------------------------- |
| `openbao_url`     | `http://127.0.0.1:30200`       | OpenBao API base URL                                          |
| `openbao_token`   | `{{ openbao_dev_root_token }}` | OpenBao root authentication token                             |
| `openbao_secrets` | List of secrets                | List of items containing `path` and `data` dictionary to seed |
