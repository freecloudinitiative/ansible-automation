# k3s-bootstrap

An Ansible role that manages the generation and storage of the K3s bootstrap token. This token is used to join nodes to the K3s cluster.

## Requirements

- OpenSSL must be installed on the target host (used for token generation).
- User with `become` (sudo) privileges.

## Features

- **Idempotent Token Check**: Verifies if a token already exists at `/etc/k3s-bootstrap-token` before performing any action.
- **Secure Generation**: Generates a cryptographically strong 64-character hexadecimal token if one is missing.
- **Permission Management**: Ensures the token file is created with `0600` permissions (read/write only for owner) to maintain security.

## What it does

1.  Checks for the existence of `/etc/k3s-bootstrap-token`.
2.  If missing, executes `openssl rand -hex 32` to generate a new token.
3.  Writes the token to `/etc/k3s-bootstrap-token`.
4.  Sets the file mode to `0600`.

## Role Variables

Currently, this role does not expose any variables in `defaults/main.yml`. It uses the fixed path:

- `/etc/k3s-bootstrap-token`: Destination path for the bootstrap token.

## Dependencies

- None.

## Example Playbook

```yaml
- hosts: masters
  become: true
  roles:
    - role: k3s-bootstrap
```

## License

MIT
