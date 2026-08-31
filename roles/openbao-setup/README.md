# Ansible Role: openbao-setup

Installs OpenBao itself — namespace, TLS material, and the Helm release — and
waits for its pods to exist. Runs before `argocd-bootstrap`, so OpenBao is
already up by the time ArgoCD applies `root-app`.

This role does **not** initialize, unseal, or configure OpenBao — that stays
`openbao-secrets-init`'s job, which runs immediately after this role.

## Why this exists

OpenBao used to be deployed by ArgoCD (sync-wave 2), and only initialized /
configured by `openbao-secrets-init` much later — after `argocd-bootstrap`.
That left a structural race: ArgoCD started syncing `external-secrets-config`
(the `ClusterSecretStore` and every `ExternalSecret`) the moment `root-app`
applied, but OpenBao's Kubernetes auth backend wasn't configured until
Ansible got to it. The `ClusterSecretStore` failed validation on the first
attempts, no `ExternalSecret` applied, and every pod needing a secret sat in
`FailedMount` until ArgoCD's automatic retry/selfHeal eventually converged a
few minutes later.

Installing OpenBao here, before ArgoCD ever runs, makes that race
structurally impossible: `external-secrets-config` finds a fully configured
OpenBao on its very first sync.

## What it does

1. Creates the `openbao` namespace with the same restricted Pod Security
   Admission labels the old GitOps-managed namespace had.
2. If `openbao-server-tls` doesn't already exist, generates a self-signed
   ECDSA P-256 certificate (no cert-manager available yet — it's still
   GitOps-managed and hasn't synced) and stores it as a `kubernetes.io/tls`
   Secret with `tls.crt` / `tls.key` / `ca.crt` (self-signed, so `ca.crt` ==
   `tls.crt`). SANs match what the Helm chart's raft `retry_join` config and
   `ClusterSecretStore`'s `caProvider` expect.
3. Fetches `infrastructure/openbao/values.yaml` from `k3s-manifests` (raw
   GitHub URL) — kept there as the single source of truth, not duplicated
   here.
4. `helm install`/`upgrade`s the `openbao/openbao` chart with those values.
5. Waits for every `openbao-N` pod to reach the `Running` phase (not `Ready`
   — OpenBao's readiness probe reflects its seal status, so pods can't be
   `Ready` until `openbao-secrets-init` unseals them; waiting for `Ready`
   here would deadlock).

## Required Variables

None — all defaults are self-contained. Override `openbao_helm_values_url` if
values ever need to come from somewhere other than `k3s-manifests` `main`.

| Variable | Default | Description |
|---|---|---|
| `openbao_namespace` | `openbao` | Namespace for the release |
| `openbao_version` | `0.28.6` | Chart version — keep in sync with the old `infrastructure/openbao/app.yaml`'s `targetRevision` |
| `openbao_helm_values_url` | k3s-manifests `main` raw URL | Where to fetch Helm values from |
