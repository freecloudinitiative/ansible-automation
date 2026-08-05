# k9s-setup

Installs [K9s](https://k9scli.io/) (terminal UI for Kubernetes clusters) onto target hosts.

## Requirements

None.

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `k9s_version` | `"v0.32.7"` | K9s release version to download |
| `k9s_install_dir` | `"/usr/local/bin"` | Directory where the binary is installed |
| `k9s_arch_map` | `{ x86_64: amd64, aarch64: arm64, armv7l: arm }` | Architecture mapping dictionary |

## Dependencies

None.

## Example Playbook

```yaml
- hosts: masters[0]
  become: true
  roles:
    - k9s-setup
```
