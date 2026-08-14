# Ansible Role: openbao-setup

Deploys OpenBao as a private, TLS-enabled three-member Raft cluster. Data and
audit logs use retained persistent volumes. Dev mode, NodePort access, the agent
injector, and the CSI provider are disabled.

## Requirements

- A working k3s cluster and `kubernetes.core` collection.
- cert-manager and a Ready `ca-cluster-issuer` ClusterIssuer.
- At least three schedulable nodes for normal HA placement.

## Important initialization step

The role deliberately does not initialize or unseal OpenBao. After the first
deployment, use a local port-forward and perform the OpenBao initialization
ceremony. Store recovery material outside the cluster and unseal the members.

```sh
kubectl -n openbao port-forward svc/openbao 8200:8200
source /etc/profile.d/openbao.sh
bao operator init
```

For a real production deployment, configure a supported auto-unseal mechanism
before launch. Never commit recovery keys or a root token.

Relevant variables are in `defaults/main.yml`, including storage sizes, chart
version, certificate issuer, and CLI version.
