# CRDs Setup Role

Ansible role to pre-install Custom Resource Definitions (CRDs) and foundational manifests for **cert-manager**, **kyverno**, and **MetalLB** on the primary Kubernetes master node.

## Role Variables

Defined in `defaults/main.yml`:

- `crds_setup_kubeconfig`: Path to the kubeconfig file (default: `/etc/rancher/k3s/k3s.yaml`).
- `crds_setup_manifests`: List of manifest objects specifying `name`, `url`, and optional `server_side` apply setting.

## Dependencies

Requires `kubernetes.core` collection.

## Example Usage

```yaml
- name: Install CRDs and Base Infrastructure Manifests
  hosts: "{{ groups['masters'][0] }}"
  become: true
  roles:
    - crds-setup
```
