# cilium-install

Installs Cilium CNI with WireGuard encryption on K3s cluster.

## Description

Installs Cilium with:

- WireGuard kernel module (all nodes)
- WireGuard encryption enabled
- Strict kube-proxy replacement
- Hubble relay and UI for observability

## Requirements

- Ansible 2.9+
- K3s cluster with `--flannel-backend=none`
- Helm installed on master
- `cilium` CLI installed on master

## Variables

| Variable                  | Description         | Default         |
| ------------------------- | ------------------- | --------------- |
| `cilium_k8s_service_host` | K8s API host        | `master1.local` |
| `cilium_k8s_service_port` | K8s API port        | `6443`          |
| `cilium_encryption_type`  | Encryption type     | `wireguard`     |
| `cilium_hubble_relay`     | Enable Hubble relay | `true`          |
| `cilium_hubble_ui`        | Enable Hubble UI    | `true`          |

## Usage

```yaml
---
- name: Setup Cilium
  hosts: masters
  become: true
  roles:
    - cilium-install
```

Run:

```bash
ansible-playbook -i inventory site.yml
```

## Tasks

1. Install wireguard-tools on all nodes
2. Check if Cilium already installed
3. Add Cilium Helm repository
4. Install Cilium with WireGuard encryption
5. Wait for Cilium pods to be ready
6. Verify encryption status
7. Assert encryption is enabled

## Verify

```bash
ssh master1.local "cilium encryption status"
ssh master1.local "kubectl get pods -n kube-system | grep cilium"
```

## License

MIT
