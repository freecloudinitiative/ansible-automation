# k3s-worker-install

Installs K3s agent on worker nodes and joins them to the K3s cluster.

## Description

Installs K3s agent using token from `/etc/rancher/node/password`, verifies readiness, and securely removes the password file post-install.

## Requirements

- Ansible 2.9+
- SSH key-based authentication
- `/etc/rancher/node/password` file (from `distribute-node-password` role)
- Master node reachable at `master1.local:6443`

## Variables

| Variable            | Description         | Default                      |
| ------------------- | ------------------- | ---------------------------- |
| `k3s_master_url`    | K3s master endpoint | `https://master1.local:6443` |
| `k3s_password_file` | Token file path     | `/etc/rancher/node/password` |
| `k3s_kubelet_port`  | Kubelet port        | `10250`                      |

## Usage

```yaml
---
- name: Setup K3s workers
  hosts: workers
  become: true
  roles:
    - distribute-node-password
    - k3s-worker-install
```

Run:

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass
```

## Tasks

1. Get node hostname
2. Download k3s install script
3. Check if k3s already installed
4. Install k3s agent with token file
5. Wait for kubelet to be ready
6. Verify installation
7. Securely remove password file
8. Assert cleanup success

## Security

- Uses `--token-file` (token never in env/logs)
- Password file removed with `shred -n 3`
- Kubelet verification ensures readiness
- Idempotent and safe to re-run

## Verify

On master:

```bash
ssh master1.local "sudo /usr/local/bin/k3s kubectl get nodes"
```

## License

MIT
