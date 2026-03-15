# Ansible Role: kyverno-setup

An Ansible role that installs the Kyverno Policy Engine for Kubernetes using its official Helm chart. This role supports specifying replica counts, applying node selectors, and ensuring both the deployment and validation webhooks are ready before finishing.

## Requirements

- `kubernetes.core` Ansible collection must be installed.
- Helm must be installed and initialized on the control node.
- A valid kubeconfig file to connect to the Kubernetes cluster.

## Role Variables

Variables available for configuration, along with their default values (see `defaults/main.yml`):

| Variable                 | Default Value                       | Description                                               |
| ------------------------ | ----------------------------------- | --------------------------------------------------------- |
| `kyverno_repo_name`      | `kyverno`                           | The name of the Kyverno Helm repository.                  |
| `kyverno_repo_url`       | `https://kyverno.github.io/kyverno` | URL to the Kyverno Helm repository.                       |
| `kyverno_namespace`      | `kyverno`                           | The Kubernetes namespace where Kyverno will be installed. |
| `kyverno_release_name`   | `kyverno`                           | The Helm release name for the Kyverno deployment.         |
| `kyverno_chart`          | `kyverno/kyverno`                   | The Helm chart reference for Kyverno.                     |
| `kyverno_replica_count`  | `1`                                 | The number of replicas for Kyverno.                       |
| `kyverno_node_selector`  | `{'node-tier': 'mid-memory'}`       | Node selector applied to the Kyverno deployment.          |
| `kyverno_timeout`        | `5m`                                | Timeout for the Kyverno Helm installation wait condition. |
| `kyverno_wait_condition` | `"stat.readyReplicas >= 1"`         | The condition used to determine if Kyverno is ready.      |
| `kubeconfig_path`        | `~/.kube/config`                    | Path to the kubeconfig file used.                         |
| `context`                | `""`                                | Specific Kubernetes context to use (optional).            |
