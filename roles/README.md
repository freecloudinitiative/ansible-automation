# Ansible Roles Catalog

This directory contains Ansible roles used to provision, configure, and manage the **Free Cloud Initiative** infrastructure. The playbooks orchestrate these roles to configure Raspberry Pi boot settings, set up a K3s Kubernetes cluster, and deploy crucial cloud-native infrastructure components.

## Infrastructure & OS Management Roles

These roles handle initial hardware/OS configuration and node health checks.

| Role | Target Hosts | Description |
|---|---|---|
| [`raspberry-pi-boot-config`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/raspberry-pi-boot-config) | `all` / Raspberry Pi nodes | Configures boot options specific to Raspberry Pi nodes, such as enabling container features in `/boot/cmdline.txt` (e.g., `cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory`). |
| [`rpi-thermal-check`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/rpi-thermal-check) | `all` / Raspberry Pi nodes | Monitors and logs CPU temperatures on Raspberry Pi nodes to prevent overheating and thermal throttling. |

---

## K3s Cluster Setup Roles

These roles manage the lifecycle of the K3s Kubernetes cluster, from initial node preparation to joining worker nodes.

| Role | Target Hosts | Description |
|---|---|---|
| [`k3s-pre-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/k3s-pre-setup) | `all` | Prepares target hosts for K3s installation. Disables swap, configures necessary kernel modules, installs prerequisite system packages, and prepares the OS environment. |
| [`k3s-master-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/k3s-master-setup) | `masters` | Installs, configures, and starts the K3s control plane on master nodes. |
| [`k3s-fact-gathering`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/k3s-fact-gathering) | `masters` | Gathers runtime facts from the master node (such as the K3s node token and master IP) so worker nodes can use them to join the cluster. |
| [`k3s-worker-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/k3s-worker-setup) | `workers` | Installs K3s agent and registers worker nodes to the cluster using the gathered master IP and token. |
| [`k3s-node-labeling`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/k3s-node-labeling) | `masters` | Post-join tasks to label and taint nodes dynamically based on inventory variables. |

---

## Core Cluster Services & GitOps

Services deployed on top of K3s to provide networking, certs, and deployment automation.

| Role | Target Hosts | Description |
|---|---|---|
| [`metallb-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/metallb-setup) | `masters` | Deploys and configures MetalLB to provide bare-metal LoadBalancer services utilizing a specified IP address pool. |
| [`cert-manager-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/cert-manager-setup) | `masters` | Installs cert-manager to manage SSL/TLS certificates and configures ClusterIssuers (e.g., Let's Encrypt). |
| [`argocd-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/argocd-setup) | `masters` | Installs ArgoCD for GitOps-driven deployment and synchronization of Kubernetes manifests and applications. |

---

## Observability & Security Stack

These roles deploy and configure telemetry collection, storage, visualization, and cluster policies.

| Role | Target Hosts | Description |
|---|---|---|
| [`prometheus-stack-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/prometheus-stack-setup) | `masters` | Deploys Prometheus and Grafana (via kube-prometheus-stack Helm chart) to monitor cluster components and workloads. |
| [`loki-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/loki-setup) | `masters` | Installs Grafana Loki for log aggregation across all Kubernetes pods and system logs. |
| [`tempo-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/tempo-setup) | `masters` | Installs Grafana Tempo for distributed tracing and request lifecycle analysis. |
| [`alloy-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/alloy-setup) | `masters` | Deploys Grafana Alloy to collect metrics, logs, and traces, forwarding them to Loki, Tempo, and Prometheus. |
| [`open-telemetry-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/open-telemetry-setup) | `masters` | Installs the OpenTelemetry Collector to build flexible, high-performance telemetry processing pipelines. |
| [`kyverno-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/kyverno-setup) | `masters` | Installs Kyverno to define and enforce security, validation, mutation, and generation policies across the cluster. |
| [`linkerd-setup`](file:///Users/entelektuelmaganda/Repositories/freecloudinitiative/ansible-automation/roles/linkerd-setup) | `masters` | Deploys Linkerd service mesh to secure pod-to-pod communications, provide mTLS, and collect golden metrics. |

---

## Standard Role Structure

Each role in this directory generally follows the standard Ansible structure:
```text
roles/<role-name>/
├── defaults/
│   └── main.yml      # Default variables (low precedence)
├── tasks/
│   └── main.yml      # Core task definitions
├── templates/
│   └── *.j2          # Jinja2 templates (configs, manifests)
└── README.md         # Role-specific documentation and usage
```
