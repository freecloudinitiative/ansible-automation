# k3s-bootstrap

This role manages the creation and storage of the K3s bootstrap token.

## What it does

- It checks if a bootstrap token already exists at `/etc/k3s-bootstrap-token`.
- It generates a random 64-character hexadecimal token if no file is found.
- It securely saves the token to `/etc/k3s-bootstrap-token` with restricted permissions (0600) for the root user.
