# k8s-node-labeling

Labels and taints K8s nodes for workload distribution and resource management.

## Description

Configures node labels and taints:

- **Labels**: Tier-based (high/mid/low memory)
- **Taints**: Workload restrictions (master, low-memory)
- **Longhorn**: Excludes low-memory node from storage

## Requirements

- Ansible 2.9+
- K3s/K8s cluster operational
- `kubectl` accessible on master

## Variables

| Variable             | Description             | Default      |
| -------------------- | ----------------------- | ------------ |
| `node_labels`        | Label configuration     | See defaults |
| `node_taints`        | Taint configuration     | See defaults |
| `longhorn_exclusion` | Longhorn disk exclusion | See defaults |

## Usage

```yaml
---
- name: Configure node labels and taints
  hosts: masters
  roles:
    - k8s-node-labeling
```

Run:

```bash
ansible-playbook -i inventory site.yml
```

## Tasks

1. Label high-memory nodes (worker5, worker6)
2. Label mid-memory nodes (worker1, worker2, worker4)
3. Label low-memory node (worker3)
4. Taint low-memory node (NoSchedule)
5. Taint master node (NoSchedule)
6. Exclude low-memory node from Longhorn
7. Display final node configuration

## Node Configuration

**High Memory (8GB):**

- `node-tier=high-memory`
- No taints

**Mid Memory (4-6GB):**

- `node-tier=mid-memory`
- No taints

**Low Memory (4GB):**

- `node-tier=low-memory`
- `memory=limited:NoSchedule`
- `node.longhorn.io/create-default-disk=false`

**Master:**

- `node-role.kubernetes.io/master=:NoSchedule`

## Verify

```bash
ssh master1.local "kubectl get nodes --show-labels"
ssh master1.local "kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints"
```

## Usage in Pod Specs

**High-memory workloads:**

```yaml
nodeSelector:
  node-tier: high-memory
```

**Avoid low-memory nodes:**

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: node-tier
              operator: NotIn
              values:
                - low-memory
```

## License

MIT
