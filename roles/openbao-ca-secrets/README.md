# Ansible Role: openbao-ca-secrets

Seeds the shared cluster CA (cert-manager's `selfsigned-ca-secret`) into the
three OpenBao paths that need it as a `ca-cert` field: `valkey`, `garage`,
`platform-postgresql`.

## Why this is a separate role

`openbao-secrets-init` now runs *before* `argocd-bootstrap`, so cert-manager
(GitOps-managed, and the creator of `selfsigned-ca-secret`) doesn't exist yet
by the time it runs. This role runs after `argocd-bootstrap` instead, once
cert-manager has had a chance to sync — still fully automatic within the same
`ansible-playbook` invocation, not a manual second run.

## What it does

1. Re-discovers OpenBao's active endpoint (reuses
   `openbao-secrets-init`'s `discover-active-endpoint.yml` via
   `include_role`/`tasks_from`, rather than trusting stale facts from
   earlier in the run — OpenBao's raft leader can in principle change
   between then and now).
2. Fetches `selfsigned-ca-secret` from the `cert-manager` namespace, waiting
   up to 150s for cert-manager to have synced and issued it.
3. `PATCH`es (`application/merge-patch+json`) just the `ca-cert` field into
   each of the three paths, so the rest of what `openbao-secrets-init`
   already wrote there isn't clobbered.

## Requires

`openbao_bootstrap_token` set as a fact — already true by this point in
`playbook.yml`, since `openbao-secrets-init` set it earlier in the same play.
