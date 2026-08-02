# OpenTelemetry Setup Role

This role automates the deployment of OpenTelemetry Collector via Helm on Kubernetes/K3s.

## Requirements

- `kubernetes.core` Ansible collection
- `helm` CLI installed on target system
- Active Kubernetes cluster with kubeconfig access

## Role Variables

Defined in `defaults/main.yml`:

| Variable | Default Value | Description |
| --- | --- | --- |
| `prometheus_namespace` | `monitoring` | Target namespace |
| `otel_stack_dir` | `~/k3s-stack/monitoring` | Directory where `otel-values.yaml` will be rendered |
| `otel_release_name` | `opentelemetry` | Helm release name |
| `otel_helm_repo_url` | `https://open-telemetry.github.io/opentelemetry-helm-charts` | OpenTelemetry Helm repository URL |
| `otel_kubeconfig` | `/etc/rancher/k3s/k3s.yaml` | Path to kubeconfig |
| `otel_port_forward_enabled` | `true` | Enable port-forwarding for the OpenTelemetry service |
| `otel_port_forward_address` | `127.0.0.1` | Local address to bind for port-forwarding |
| `otel_port_forward_local_port` | `4317` | Local port for port-forwarding |
| `otel_port_forward_remote_port` | `4317` | Remote service port |

> [!NOTE]
> `otel_service_name` is discovered dynamically at runtime by the "Set otel_service_name from discovered service" task and is overwritten during role execution rather than being set in `defaults/main.yml`.

## Example Playbook

```yaml
- hosts: master
  roles:
    - open-telemetry-setup
```
