# ansible-automation

## What Code Do

`playbook.yml` builds the FCI K3s cluster on bare-metal nodes, then hands the cluster to GitOps.

Three plays, in order:

1. **Masters** — `k3s-pre-setup` then `k3s-master-setup` then `k3s-fact-gathering`. First host in `masters` runs `k3s server --cluster-init`. Traefik and ServiceLB stay off (GitOps owns ingress and LB). Embedded registry on. Kubeconfig copied to `~/.kube/config`. Helm, kubectx, kubens, popeye, stern, kube-ps1 installed. Node token slurped from `/var/lib/rancher/k3s/server/node-token`.
2. **Workers** — `k3s-pre-setup` then `k3s-worker-setup`. Join `https://{{ k3s_master1_public_ip }}:6443` with that token.
3. **First master only** — `k3s-node-labeling`, `argocd-setup`, `argocd-bootstrap`, `openbao-secrets-init`. Labels `node-tier`. Taints `low_memory` and masters. Installs Argo CD from official manifest. Applies `root-app` pointing at `k3s-manifests`. Seeds OpenBao after operator init/unseal.

OpenBao install is **not** in this playbook. `k3s-manifests` deploys it. `openbao-secrets-init` waits, then exits unless OpenBao is initialized **and** `OPENBAO_BOOTSTRAP_TOKEN` is set.

```bash
# First run: cluster + Argo CD + root-app. OpenBao seed stops until unsealed.
ansible-playbook playbook.yml --ask-vault-pass

# After OpenBao init/unseal: seed KV, Kubernetes auth, External Secrets policy.
OPENBAO_BOOTSTRAP_TOKEN='hvs....' \
AUTHENTIK_SECRET_KEY='...' \
AUTHENTIK_POSTGRESQL_PASSWORD='...' \
PLATFORM_POSTGRESQL_PASSWORD='...' \
AUTHENTIK_BOOTSTRAP_PASSWORD='...' \
VALKEY_PASSWORD='...' \
ansible-playbook playbook.yml --ask-vault-pass
```

`ansible.cfg` sets `inventory = inventory.ini` and `ask_vault_pass = true`. Revoke bootstrap token after seed.

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

ssh_config_default_user
ssh_config_default_identity_file
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

authentik_bootstrap_email
authentik_bootstrap_password
authentik_postgresql_password
authentik_secret_key
authentik_grafana_oidc_secret
authentik_argocd_oidc_secret
authentik_zot_oidc_secret

cloudflared_tunnel_token
compute_postgresql_password
database_postgresql_password

garage_storage_service_access_key
garage_storage_service_secret_key

iam_postgresql_password
platform_postgresql_password
storage_postgresql_password

terminal_gateway_internal_public_key
terminal_gateway_internal_signing_key

valkey_password

zot_registry_htpasswd
zot_registry_pull_password
zot_registry_pull_username
zot_registry_s3_access_key_id
zot_registry_s3_secret_access_key
```

## Folder Where

```
playbook.yml                 Cluster bootstrap. Only playbook this doc covers.
ansible.cfg                  Inventory, vault prompt, SSH multiplexing.
inventory.ini                masters / workers / high_memory / mid_memory / low_memory.
group_vars/all/              Shared vars. secret.yml is Ansible Vault.
collections/requirements.yml kubernetes.core and friends.
roles/                       Roles playbook.yml calls. See FILES.md.
```

## Read More Where

- [ROLES.md](ROLES.md) — plays, roles, handoff to GitOps
- [FILES.md](FILES.md) — every playbook-related file, one line
