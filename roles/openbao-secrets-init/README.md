# Ansible Role: openbao-secrets-init

Runs immediately after `openbao-setup` (which installs OpenBao itself), still
before `argocd-setup`/`argocd-bootstrap`. Performs an idempotent,
post-initialization OpenBao bootstrap: enables KV v2, persistent audit
logging, Kubernetes authentication, a narrow read-only policy, and the
initial Grafana, Authentik, PostgreSQL, and Valkey secrets — every seeded
path except `valkey`/`garage`/`platform-postgresql`'s `ca-cert` field, which
needs cert-manager's CA and is seeded later by `openbao-ca-secrets` (see that
role's README). Doing OpenBao's init/unseal/seed here, before ArgoCD ever
runs, is what lets `external-secrets-config`'s `ClusterSecretStore` sync
clean on its very first attempt instead of racing ArgoCD's startup.

The role never writes an OpenBao token into Kubernetes. External Secrets uses a
short-lived Kubernetes ServiceAccount token to authenticate to OpenBao.

Bootstrap API calls are made directly to the selected OpenBao EndpointSlice
address and delegated to the Ansible inventory host that owns that endpoint.
K3s sets each Kubernetes node name from `inventory_hostname`, so the request
originates on the OpenBao pod's local node. This preserves the restrictive
OpenBao NetworkPolicy without granting every cluster node or a public ingress
bootstrap access. The role also waits for a serving, non-terminating
`openbao-active` endpoint before performing authenticated writes, rather than
treating creation of the Service object—or a stale EndpointSlice entry—as proof
that an active server is reachable. Health and active-endpoint retries refresh
EndpointSlices and probe every eligible candidate on each attempt, so a replaced
Pod does not leave the bootstrap loop pinned to an obsolete Pod IP. Both gates
share one 600-second discovery deadline; individual Pod probes have a three-second
socket timeout, so blackholed Pod IPs cannot silently expand the total wait by
many minutes.

Supply a short-lived administrative bootstrap token, and every other secret
below, through the process environment for the single `ansible-playbook` run
— `openbao-setup` installs OpenBao and `openbao-secrets-init` initializes and
unseals it within that same run, so there's no separate "after init/unseal"
step to wait for:

```sh
GHCR_REGISTRY_USERNAME='github_username' \
GHCR_REGISTRY_TOKEN='ghp_random_token_with_read_packages_scope' \
AUTHENTIK_SECRET_KEY='at-least-50-random-characters' \
AUTHENTIK_POSTGRESQL_PASSWORD='at-least-24-random-characters' \
PLATFORM_POSTGRESQL_PASSWORD='at-least-24-random-characters' \
AUTHENTIK_BOOTSTRAP_PASSWORD='at-least-16-random-characters' \
AUTHENTIK_ADMIN_TOKEN='at-least-32-random-characters' \
VALKEY_PASSWORD='at-least-24-random-characters' \
OPENBAO_BOOTSTRAP_TOKEN='hvs.example' \
ansible-playbook playbook.yml
```

Revoke that token when the bootstrap succeeds. `no_log` protects every task
that handles it, but shell history and process environments must still be
treated as sensitive.

Generate every value independently with a cryptographically secure password
manager. `AUTHENTIK_BOOTSTRAP_EMAIL` is optional and defaults to
`admin@freecloudinitiative.com`. After the first Authentik login, change the
bootstrap password and enable MFA for the administrator.

`AUTHENTIK_ADMIN_TOKEN` is not read *from* Authentik — it is a value you pick,
seeded into OpenBao like any other secret. It is also fed to Authentik itself
as `AUTHENTIK_BOOTSTRAP_TOKEN`, which makes Authentik create the akadmin API
token with that exact value at boot. Both sides converge on the same value
without a manual UI step or a second `ansible-playbook` run.
