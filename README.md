# ansible-automation

## What Code Do

`playbook.yml` builds the FCI K3s cluster on bare-metal nodes, then hands the cluster to GitOps.

Six plays, in order:

1. **All hosts** — inventory validation, cgroups, `k3s-pre-setup`.
2. **Masters** — `k3s-master-setup` then `k3s-fact-gathering`. First host in `masters` runs `k3s server --cluster-init`. Traefik and ServiceLB stay off (GitOps owns ingress and LB). Embedded registry on. Kubeconfig copied to `~/.kube/config`. Helm, kubectx, kubens, popeye, stern, kube-ps1 installed. Node token slurped from `/var/lib/rancher/k3s/server/node-token`.
3. **Workers** — `k3s-worker-setup`. Join `https://{{ k3s_master1_public_ip }}:6443` with that token.
4. **high_memory workers** — `kata-containers`. Official static tarball into `/opt/kata`. Registers CRI runtime `kata` with k3s containerd. Applies RuntimeClass `kata` (`node-tier=high-memory`). Needs `/dev/kvm` (nested virt if the worker is a VM). ~3 GB disk. Re-run with `--tags kata`.
5. **First master only** — `k3s-node-labeling`, `argocd-setup`, `argocd-bootstrap`, `openbao-secrets-init`. Labels `node-tier`. Taints `low_memory` and masters. Installs Argo CD from official manifest. Applies `root-app` pointing at `k3s-manifests`. Seeds OpenBao after operator init/unseal.
6. **Local k9s** — `local-k9s-setup`. Fetch kubeconfig and configure k9s on the control node.

OpenBao install is **not** in this playbook. `k3s-manifests` deploys it. `openbao-secrets-init` waits, then exits unless OpenBao is initialized **and** `OPENBAO_BOOTSTRAP_TOKEN` is set.

## Prerequisites

Before running `ansible-playbook playbook.yml`, set `k3s_master1_public_ip` to the public IP of the first master node. Workers and secondary masters join the cluster at:

```text
--server=https://<k3s_master1_public_ip>:6443
```

Set it in `group_vars/all/main.yml`:

```yaml
k3s_master1_public_ip: "203.0.113.10"
```

Or pass it at runtime:

```bash
ansible-playbook playbook.yml -e k3s_master1_public_ip=203.0.113.10 --ask-vault-pass
```

The playbook validates this value before modifying any node. An empty or whitespace-only value is rejected at preflight; without this guard an unusable endpoint such as `https://:6443` would only surface as a join failure after k3s was already installed on the masters.

```bash
# First run: cluster + Argo CD + root-app. OpenBao seed stops until unsealed.
ansible-playbook playbook.yml --ask-vault-pass

# After OpenBao init/unseal: seed KV, Kubernetes auth, External Secrets policy.
# Every variable below is asserted before any OpenBao write; an unset variable
# would seed an empty string and surface as a CrashLoopBackOff or
# ImagePullBackOff with no pointer back at this run.
OPENBAO_BOOTSTRAP_TOKEN='hvs....' \
GRAFANA_ADMIN_PASSWORD='...' \
AUTHENTIK_SECRET_KEY='...' \
AUTHENTIK_POSTGRESQL_PASSWORD='...' \
AUTHENTIK_BOOTSTRAP_PASSWORD='...' \
AUTHENTIK_GRAFANA_OIDC_SECRET='...' \
AUTHENTIK_ARGOCD_OIDC_SECRET='...' \
PLATFORM_POSTGRESQL_PASSWORD='...' \
VALKEY_PASSWORD='...' \
IAM_POSTGRESQL_PASSWORD='...' \
COMPUTE_POSTGRESQL_PASSWORD='...' \
DATABASE_POSTGRESQL_PASSWORD='...' \
STORAGE_POSTGRESQL_PASSWORD='...' \
API_GATEWAY_INTERNAL_PUBLIC_KEY='...' \
API_GATEWAY_INTERNAL_SIGNING_KEY='...' \
TERMINAL_GATEWAY_INTERNAL_PUBLIC_KEY='...' \
TERMINAL_GATEWAY_INTERNAL_SIGNING_KEY='...' \
COMPUTE_INTERNAL_PUBLIC_KEY='...' \
COMPUTE_INTERNAL_SIGNING_KEY='...' \
DATABASE_INTERNAL_PUBLIC_KEY='...' \
DATABASE_INTERNAL_SIGNING_KEY='...' \
STORAGE_INTERNAL_PUBLIC_KEY='...' \
STORAGE_INTERNAL_SIGNING_KEY='...' \
GARAGE_STORAGE_SERVICE_ACCESS_KEY='...' \
GARAGE_STORAGE_SERVICE_SECRET_KEY='...' \
GHCR_REGISTRY_USERNAME='...' \
GHCR_REGISTRY_TOKEN='...' \
CLOUDFLARED_TUNNEL_TOKEN='...' \
ansible-playbook playbook.yml --ask-vault-pass
```

Minimum lengths and format requirements enforced by the assert task before any write:

| Variable | Requirement |
|---|---|
| `AUTHENTIK_SECRET_KEY` | >= 50 chars |
| `AUTHENTIK_POSTGRESQL_PASSWORD`, `PLATFORM_POSTGRESQL_PASSWORD`, `VALKEY_PASSWORD` | >= 24 chars |
| `IAM_POSTGRESQL_PASSWORD`, `COMPUTE_POSTGRESQL_PASSWORD`, `DATABASE_POSTGRESQL_PASSWORD`, `STORAGE_POSTGRESQL_PASSWORD` | >= 24 chars |
| `AUTHENTIK_BOOTSTRAP_PASSWORD`, `AUTHENTIK_GRAFANA_OIDC_SECRET`, `AUTHENTIK_ARGOCD_OIDC_SECRET`, `GRAFANA_ADMIN_PASSWORD` | >= 16 chars |
| `AUTHENTIK_ADMIN_TOKEN` | >= 32 chars |
| `*_INTERNAL_PUBLIC_KEY` (5 keys) | valid PEM `BEGIN PUBLIC KEY`, distinct across all issuers |
| `*_INTERNAL_SIGNING_KEY` (5 keys) | valid PEM `BEGIN PRIVATE KEY` |
| `GARAGE_STORAGE_SERVICE_ACCESS_KEY`, `GARAGE_STORAGE_SERVICE_SECRET_KEY` | >= 16 chars |
| `GHCR_REGISTRY_TOKEN` | >= 16 chars |
| `GHCR_REGISTRY_USERNAME` | non-empty |
| `CLOUDFLARED_TUNNEL_TOKEN` | >= 32 chars |

- `GRAFANA_ADMIN_PASSWORD`: May instead live in `group_vars/all/secret.yml`. Note that `group_vars` overrides the role default.

`ansible.cfg` sets `inventory = inventory.ini` and `ask_vault_pass = true`. Revoke bootstrap token after seed.

## Single-run bootstrap

`AUTHENTIK_ADMIN_TOKEN` is a value **you pick** — a random string, generated
the same way as every other secret — not a token pulled out of Authentik. It
is seeded into OpenBao like everything else, and also passed through to
Authentik itself as `AUTHENTIK_BOOTSTRAP_TOKEN`
(`k3s-manifests/infrastructure/external-secrets/external-secret-authentik.yaml`),
which makes Authentik create the akadmin API token with that exact value at
boot (sync-wave 5). Both sides converge on the same value without a manual
UI step or a second `ansible-playbook` run:

```bash
AUTHENTIK_ADMIN_TOKEN='at-least-32-random-characters' \
OPENBAO_BOOTSTRAP_TOKEN='hvs....' \
  ansible-playbook playbook.yml --ask-vault-pass
```

External Secrets refreshes `iam-service-authentik-token` within `refreshInterval`
(1 h by default); force it sooner:

```bash
kubectl -n backend annotate externalsecret iam-service-authentik-token \
  force-sync=$(date +%s) --overwrite
```

No iam-service restart is needed — the token file is re-read per call, not
cached at startup.

## Why Need It

- K3s on Raspberry Pi nodes has no cloud installer. Swap, packages, server/agent flags, token handoff must be exact.
- Argo CD is the cutover: this repo stops at `root-app`. `k3s-manifests` owns Traefik, cert-manager, OpenBao, apps.
- Secrets never land in Git as plaintext. Vault file + env vars + OpenBao. External Secrets reads OpenBao with a short-lived ServiceAccount token. Playbook never writes an OpenBao token into Kubernetes.

## Security

`group_vars/all/secret.yml` is tracked in Git as an Ansible Vault-encrypted blob. Never decrypt it and commit the result.

**Activate the pre-commit hook once after cloning:**

```bash
git config core.hooksPath hooks
```

The hook (`hooks/pre-commit`) inspects the **staged** content of any `secret.yml` file and refuses the commit if the file does not start with `$ANSIBLE_VAULT`. CI runs the same check as a backstop.

### Vault Parameters

Parameters in `group_vars/all/secret.yml`:

```yaml
password
kubeconfig_path
traefik_admin_password
grafana_admin_password
openbao_dev_root_token

ssh_config_gcp_user
ssh_config_gcp_identity_file
ssh_config_aws_user
ssh_config_aws_identity_file
ssh_config_custom_hosts:
  - name
    hostname
    user
    identity_file
    identities_only
    add_keys_to_agent
    use_keychain

api_gateway_internal_public_key
api_gateway_internal_signing_key

authentik_admin_token
authentik_bootstrap_email
authentik_bootstrap_password
authentik_postgresql_password
authentik_secret_key
authentik_grafana_oidc_secret
authentik_argocd_oidc_secret

cloudflared_tunnel_token
compute_postgresql_password
compute_internal_public_key
compute_internal_signing_key
database_postgresql_password
database_internal_public_key
database_internal_signing_key

garage_storage_service_access_key
garage_storage_service_secret_key

iam_postgresql_password
platform_postgresql_password
storage_postgresql_password
storage_internal_public_key
storage_internal_signing_key

terminal_gateway_internal_public_key
terminal_gateway_internal_signing_key

valkey_password
ghcr_registry_username
ghcr_registry_token
```

### Generating Keypairs for Services

The api-gateway, terminal-gateway, compute, database, and storage services each require their own distinct Ed25519 keypair for internal communication. All five keypairs MUST be distinct from each other. A shared `kid` (derived from the public key) will cause the iam-service to fail at boot with a kid-collision error that names a key fingerprint, not an environment variable.

To generate all five:

```bash
for svc in api-gateway terminal-gateway compute database storage; do
  openssl genpkey -algorithm ed25519 -out "$svc-internal-signing.key"
  openssl pkey -in "$svc-internal-signing.key" -pubout -out "$svc-internal-public.pem"
done
```

Add the generated keys to `group_vars/all/secret.yml` before running the playbook, or export them as environment variables (e.g. `API_GATEWAY_INTERNAL_SIGNING_KEY`, `API_GATEWAY_INTERNAL_PUBLIC_KEY`, `TERMINAL_GATEWAY_INTERNAL_SIGNING_KEY`, `TERMINAL_GATEWAY_INTERNAL_PUBLIC_KEY`, `COMPUTE_INTERNAL_SIGNING_KEY`, `COMPUTE_INTERNAL_PUBLIC_KEY`, etc.) for runs that bypass the vault file.

## Folder Where

```
playbook.yml                 Cluster bootstrap.
ssh-config.yml               Nonprod SSH config.
thermal-check.yml            Check node temps.
ansible.cfg                  Inventory, vault prompt, SSH multiplexing.
inventory.ini                masters / workers / high_memory / mid_memory / low_memory.
group_vars/all/              Shared vars. secret.yml is Ansible Vault.
collections/requirements.yml kubernetes.core and friends.
roles/                       Roles playbook.yml calls. See FILES.md.
```

## Read More Where

- [ROLES.md](ROLES.md) — plays, roles, handoff to GitOps
- [FILES.md](FILES.md) — every playbook-related file, one line
