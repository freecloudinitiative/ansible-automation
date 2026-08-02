# cert-manager Setup Role

This role automates the deployment of [cert-manager](https://cert-manager.io) via Helm on Kubernetes/K3s,
and configures a `ClusterIssuer` for automatic TLS certificate provisioning.

## Requirements

- `kubernetes.core` Ansible collection
- `helm` CLI installed on target system
- Active K3s cluster with kubeconfig access

## Role Variables

Defined in `defaults/main.yml`:

| Variable | Default | Description |
|---|---|---|
| `cert_manager_namespace` | `cert-manager` | Namespace to install cert-manager into |
| `cert_manager_stack_dir` | `~/k3s-stack/cert-manager` | Local directory for rendered manifests |
| `cert_manager_release_name` | `cert-manager` | Helm release name |
| `cert_manager_helm_repo_url` | `https://charts.jetstack.io` | Jetstack Helm repo URL |
| `cert_manager_kubeconfig` | `/etc/rancher/k3s/k3s.yaml` | Path to kubeconfig |
| `cert_manager_issuer_type` | `selfsigned` | ClusterIssuer type: `selfsigned`, `letsencrypt-staging`, `letsencrypt-production` |
| `cert_manager_letsencrypt_email` | `""` | Required for Let's Encrypt issuers |
| `cert_manager_prometheus_enabled` | `true` | Enable Prometheus metrics endpoint |
| `cert_manager_servicemonitor_enabled` | `false` | Enable Prometheus ServiceMonitor (Requires `monitoring.coreos.com/v1` ServiceMonitor CRD) |
| `cert_manager_servicemonitor_namespace` | `monitoring` | Namespace for the ServiceMonitor resource |

> [!NOTE]
> Setting `cert_manager_servicemonitor_enabled: true` requires the Prometheus Operator (`monitoring.coreos.com/v1` ServiceMonitor CRD) to be installed in the cluster prior to deployment (e.g. via `prometheus-stack-setup`).

## ClusterIssuer Types

### `selfsigned` (default)
Creates two ClusterIssuers:
- `selfsigned-cluster-issuer` — raw self-signed
- `ca-cluster-issuer` — CA-backed issuer (recommended for internal services)

Use `ca-cluster-issuer` in your `Certificate` and `Ingress` resources.

### `letsencrypt-staging`
Uses Let's Encrypt staging API (untrusted certs, for testing). Requires:
- `cert_manager_letsencrypt_email` to be set
- HTTP01 challenge via Traefik (K3s default ingress)

### `letsencrypt-production`
Uses Let's Encrypt production API (trusted, rate-limited). Same requirements as staging.

## Example Playbook

```yaml
- hosts: masters
  roles:
    - cert-manager-setup
```

## Example — Let's Encrypt Production

```yaml
- hosts: masters
  vars:
    cert_manager_issuer_type: letsencrypt-production
    cert_manager_letsencrypt_email: admin@example.com
  roles:
    - cert-manager-setup
```

## Using the ClusterIssuer in an Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: ca-cluster-issuer
spec:
  tls:
    - hosts:
        - grafana.example.local
      secretName: grafana-tls
  rules:
    - host: grafana.example.local
      ...
```
