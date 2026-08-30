# Ansible Role: openbao-secrets-init

Performs an idempotent, post-initialization OpenBao bootstrap. It enables KV v2,
persistent audit logging, Kubernetes authentication, a narrow read-only policy,
and the initial Grafana, Authentik, PostgreSQL, and Valkey secrets.

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
Pod does not leave the bootstrap loop pinned to an obsolete Pod IP.

After OpenBao has been initialized and unsealed, rerun the playbook with a
short-lived administrative bootstrap token supplied only through the process
environment:

```sh
GHCR_REGISTRY_USERNAME='github_username' \
GHCR_REGISTRY_TOKEN='ghp_random_token_with_read_packages_scope' \
AUTHENTIK_SECRET_KEY='at-least-50-random-characters' \
AUTHENTIK_POSTGRESQL_PASSWORD='at-least-24-random-characters' \
PLATFORM_POSTGRESQL_PASSWORD='at-least-24-random-characters' \
AUTHENTIK_BOOTSTRAP_PASSWORD='at-least-16-random-characters' \
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
