# Ansible Role: sealed-secrets

An Ansible role that installs the Bitnami Sealed Secrets controller in a Kubernetes cluster using Helm and also installs the `kubeseal` CLI tool on the local machine (or the machine running Ansible).

## Requirements

- `kubernetes.core` Ansible collection must be installed.
- Helm must be installed on the target machine.
- A valid kubeconfig file to connect to the Kubernetes cluster.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable                      | Default Value                   | Description                                                             |
| ----------------------------- | ------------------------------- | ----------------------------------------------------------------------- |
| `sealed_secrets_namespace`    | `kube-system`                   | The Kubernetes namespace where Sealed Secrets will be installed.        |
| `sealed_secrets_release_name` | `sealed-secrets`                | The Helm release name for the Sealed Secrets deployment.                |
| `sealed_secrets_chart`        | `sealed-secrets/sealed-secrets` | The Helm chart reference for Sealed Secrets.                            |
| `kubeseal_install_path`       | `/usr/local/bin`                | The path where the `kubeseal` binary will be installed.                 |
| `kubeseal_tmp_dir`            | `/tmp`                          | The temporary directory used for downloading and extracting `kubeseal`. |
| `kubeconfig_path`             | `omit`                          | Path to the kubeconfig file.                                            |
| `context`                     | `omit`                          | The kubernetes context to use.                                          |
