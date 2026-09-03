# kata-containers

Installs Kata Containers on `high_memory` K3s workers and registers CRI runtime `kata`.

Each pod with `runtimeClassName: kata-qemu` boots a QEMU/KVM microVM. Do not run this role on Raspberry Pi or any host without `/dev/kvm`. Nested virtualization must be on if the worker itself is a VM.

## What it does

- Checks VT-x / AMD-V and loads `kvm` + `kvm_intel` or `kvm_amd`.
- Installs the official static tarball into `/opt/kata` (pinned QEMU, guest kernel, guest image, shim).
- Symlinks `kata-runtime` and `containerd-shim-kata-v2` onto `PATH`.
- Copies QEMU config to `/etc/kata-containers/configuration.toml`.
- Extends k3s containerd with only the matching template: `config-v3.toml.tmpl` when live `config.toml` is version 3 (containerd 2), `config.toml.tmpl` when version 2 (containerd 1.7). Does not create a v3 tmpl on a v2 node — k3s would prefer it and take the agent down.
- Restarts `k3s-agent` (or `k3s`) only when the containerd template or install changes.
- Applies RuntimeClass `kata-qemu` with `nodeSelector: node-tier=high-memory`.
- Labels the node `fci.io/runtime=kata` — the label compute-service selects dedicated instances onto.

## The two names compute-service depends on

`kata_runtimeclass_name` and `kata_node_label_key`/`_value` are a contract with compute-service,
not free choices. Its `instancetype.Checker` offers the dedicated instance type only when it finds
**both** a RuntimeClass named exactly `kata-qemu` **and** at least one schedulable, Ready node
labelled exactly `fci.io/runtime=kata`; its projection then pins dedicated pods to that same label.

Neither mismatch fails loudly. The dedicated type simply never appears in the API, with nothing in
any log to explain the absence — so if dedicated instances are missing after this role has run,
check these two names first.

| | This role applies | compute-service looks for |
| --- | --- | --- |
| RuntimeClass | `kata_runtimeclass_name` | `kataRuntimeClass`, `internal/instancetype/types.go` |
| Node label | `kata_node_label_key=kata_node_label_value` | `kataNodeSelector`, `internal/instancetype/capability.go` |

The RuntimeClass *name* is not the containerd handler. `kata_runtime_handler` names the runtime
section this role writes into containerd's config and stays `kata`; renaming the class does not
touch it.

### Not done here: the NoSchedule taint

compute-service's projection also gives dedicated pods a toleration for
`fci.io/runtime=kata:NoSchedule`, so that dedicated workloads can land on a tainted Kata pool while
ordinary instances cannot. This role labels nodes but does not taint them, so today the reverse
does not hold: shared instances can still schedule onto Kata workers. Adding the taint would evict
every platform workload on those nodes that lacks the toleration, which is a deployment decision
rather than a naming fix — do it deliberately, not as a side effect of this role.

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
| `kata_runtimeclass_name` | `kata-qemu` | RuntimeClass name. Must match compute-service's `kataRuntimeClass`; the containerd handler is `kata_runtime_handler`, which is separate |
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
