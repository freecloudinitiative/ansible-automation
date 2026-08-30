# kata-containers

Installs Kata Containers on `high_memory` K3s workers and registers CRI runtime `kata`.

Each pod with `runtimeClassName: kata` boots a QEMU/KVM microVM. Do not run this role on Raspberry Pi or any host without `/dev/kvm`. Nested virtualization must be on if the worker itself is a VM.

## What it does

- Checks VT-x / AMD-V and loads `kvm` + `kvm_intel` or `kvm_amd`.
- Installs the official static tarball into `/opt/kata` (pinned QEMU, guest kernel, guest image, shim).
- Symlinks `kata-runtime` and `containerd-shim-kata-v2` onto `PATH`.
- Copies QEMU config to `/etc/kata-containers/configuration.toml`.
- Extends k3s containerd via `config-v3.toml.tmpl` (containerd 2) and `config.toml.tmpl` (containerd 1.7).
- Restarts `k3s-agent` (or `k3s`) only when the containerd template or install changes.
- Applies RuntimeClass `kata` with `nodeSelector: node-tier=high-memory`.
- Labels the node `katacontainers.io/kata-runtime=true`.

## Requirements

- Ubuntu worker already joined as a k3s agent (`k3s-worker-setup`).
- Hardware virtualization. On a VM: nested virt enabled on the hypervisor.
- ~3 GB free disk for `/opt/kata`.
- `kubernetes.core` on the controller. RuntimeClass is applied on `groups['masters'][0]`.

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `kata_version` | `4.1.0` | GitHub release tag |
| `kata_release_artifact` | `kata-static` | `kata-static` (runtime-rs) or `kata-go-static` (Go shim). 3.x only has `kata-static` |
| `kata_default_memory_mb` | `2048` | Guest VM boot memory (MiB) |
| `kata_default_vcpus` | `1` | Guest VM boot vCPUs |
| `kata_require_kvm` | `true` | Fail if `/dev/kvm` is missing after module load |
| `kata_apply_runtimeclass` | `true` | Apply cluster RuntimeClass on `masters[0]` |
| `kata_runtimeclass_name` | `kata` | RuntimeClass name and containerd handler |
| `kata_runtimeclass_node_selector` | `{node-tier: high-memory}` | Restricts kata pods to labeled high-memory nodes |
| `kata_cleanup_tarball` | `true` | Delete the downloaded `.tar.zst` after extract |
| `kubeconfig_path` | `/etc/rancher/k3s/k3s.yaml` | Used for RuntimeClass and node label |

## Example

```yaml
- name: Install Kata Containers on high-memory workers
  hosts: high_memory
  become: true
  roles:
    - kata-containers
```

Standalone after the cluster exists:

```bash
ansible-playbook playbook.yml --tags kata --ask-vault-pass
```

## Use a Kata pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kata-smoke
spec:
  runtimeClassName: kata
  containers:
    - name: uname
      image: alpine:3.20
      command: ["uname", "-a"]
```

Guest kernel differs from the host kernel. That is the isolation boundary.

## License

MIT
