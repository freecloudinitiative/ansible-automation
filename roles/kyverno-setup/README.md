# Kyverno Setup Role

This role automates the deployment of [Kyverno](https://kyverno.io) (CNCF Incubating) via Helm on Kubernetes/K3s,
and applies a baseline set of `ClusterPolicy` resources for security and best-practice enforcement.

## Requirements

- `kubernetes.core` Ansible collection
- `helm` CLI installed on target system
- Active K3s cluster with kubeconfig access

## Role Variables

Defined in `defaults/main.yml`:

| Variable | Default | Description |
|---|---|---|
| `kyverno_namespace` | `kyverno` | Namespace to install Kyverno into |
| `kyverno_stack_dir` | `~/k3s-stack/kyverno` | Local directory for rendered manifests |
| `kyverno_release_name` | `kyverno` | Helm release name |
| `kyverno_helm_repo_url` | `https://kyverno.github.io/kyverno/` | Kyverno Helm repo URL |
| `kubeconfig` | `/etc/rancher/k3s/k3s.yaml` | Path to kubeconfig |
| `port_forward_enabled` | `true` | Enable background port-forwarding |
| `port_forward_address` | `0.0.0.0` | Listening address for port-forwarding |
| `kyverno_policy_mode` | `audit` | Policy enforcement mode: `audit` (log only) or `enforce` (block) |
| `kyverno_ui_enabled` / `ui_enable` | `true` | Enable lightweight Policy Reporter UI dashboard for Kyverno |
| `kyverno_policy_require_labels` | `true` | Enable: require `app` and `env` labels on Pods |
| `kyverno_policy_disallow_latest_tag` | `true` | Enable: disallow `:latest` image tags |
| `kyverno_policy_require_resource_limits` | `true` | Enable: require CPU/memory requests and limits |
| `kyverno_policy_disallow_privileged` | `true` | Enable: disallow privileged containers |

## Bundled ClusterPolicies

| Policy | Category | Severity | Description |
|---|---|---|---|
| `require-pod-labels` | Best Practices | Medium | Pods must have `app` and `env` labels |
| `disallow-latest-tag` | Best Practices | Medium | Reject `:latest` or untagged images |
| `require-resource-limits` | Best Practices | Medium | All containers must define CPU/memory requests and limits |
| `disallow-privileged-containers` | Pod Security | High | Detect containers running with `privileged: true` (blocks in enforce mode) |

## Policy Modes

### `audit` (default — safe for day-one rollout)
Policies log violations to Kyverno's `PolicyReport` CRDs without blocking anything.
Use this first to understand what your existing workloads violate before switching to enforce.

```bash
kubectl get policyreport -A
kubectl get clusterpolicyreport
```

### `enforce`
Non-compliant resources are rejected at admission time with a descriptive error.
Switch once violations are remediated:

```yaml
kyverno_policy_mode: enforce
```

## Example Playbook

```yaml
- hosts: masters
  roles:
    - kyverno-setup
```

## Switching to Enforce Mode

```yaml
- hosts: masters
  vars:
    kyverno_policy_mode: enforce
  roles:
    - kyverno-setup
```

## Adding a PolicyException

To exempt a specific workload from a policy without disabling the whole policy:

```yaml
apiVersion: kyverno.io/v2
kind: PolicyException
metadata:
  name: allow-monitoring-privileged
  namespace: monitoring
spec:
  exceptions:
    - policyName: disallow-privileged-containers
      ruleNames:
        - check-privileged
  match:
    any:
      - resources:
          namespaces:
            - monitoring
          names:
            - node-exporter-*
```
