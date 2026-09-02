# ROLES — How Code Work Together

## Big Picture

```
operator
  │
  │  ansible-playbook playbook.yml --ask-vault-pass
  ▼
playbook.yml
  │
  ├─ play 1  hosts: masters
  │    k3s-master-setup → k3s-fact-gathering
  │    system prerequisites, swap off, APT packages
  │    first master: k3s server --cluster-init
  │    later masters: join :6443 (inventory has only master-1 uncommented)
  │    slurp node-token → hostvars
  │
  ├─ play 2  hosts: workers
  │    k3s-worker-setup
  │    token from hostvars[groups['masters'][0]]
  │
  ├─ play 3  hosts: high_memory
  │    kata-containers
  │    QEMU/KVM runtime on high-memory workers only
  │
  ├─ play 4  hosts: groups['masters'][0]
       k3s-node-labeling
            │
            ▼
       openbao-setup ── Helm install, self-signed TLS (no cert-manager yet)
            │
            ▼
       openbao-secrets-init ── wait, init/unseal, OPENBAO_BOOTSTRAP_TOKEN,
            │                  KV secrets + kubernetes auth + ES policy
            │                  (everything except the 3 CA-dependent fields)
            ▼
       argocd-setup → argocd-bootstrap
            │
            ▼
       Argo CD Application root-app
            │
            ▼
       k3s-manifests  (infrastructure/ + applications/) ── OpenBao is NOT
            │                                              one of these
            ▼
       openbao-ca-secrets ── now that cert-manager has synced, patch
                              valkey/garage/platform-postgresql ca-cert
  │
  └─ play 5  hosts: groups['masters'][0]
       local-k9s-setup
       fetch kubeconfig, configure local k9s
```

Post-play debug prints public URLs. Passwords stay commented out.

## What Part Do

| Part | Job |
|---|---|
| `playbook.yml` | Ordered master, worker, optional Kata, cluster-component, and local tooling plays. |
| `ssh-config.yml` | Nonprod SSH config. |
| `thermal-check.yml` | Check node temps. |
| `inventory.ini` | Host groups. `masters[0]` is primary. Memory groups drive labels/taints. |
| `group_vars/all/main.yml` | `ssh_config_cloud`. `k3s_*_public_ip` slots. Empty until filled. |
| `group_vars/all/secret.yml` | Vault. `--ask-vault-pass`. |
| `ansible.cfg` | Default inventory, vault prompt, `roles_path=./roles`. |
| `collections/requirements.yml` | `kubernetes.core` for k8s modules. |

## What Roles Do

| Role | Play | Job |
|---|---|---|
| `inventory-validation` | 1 | Validate required inventory variables (`k3s_master1_public_ip`, `masters`, `workers`) before modifying any nodes. Gated HA check when multiple masters exist. |
| `k3s-master-setup` | 1 | Master prerequisites, swap off, `get.k3s.io` server, readyz wait, Helm + CLI tools. |
| `k3s-fact-gathering` | 2 | Slurp node-token on `masters[0]`. Set `kubeconfig_path`. |
| `k3s-worker-setup` | 3 | Installs missing agents only, up to five workers at a time (`throttle: 5`). Uses `--server`, `--token`, and `worker_label`; restarts and re-waits on a failed `systemctl start`. |
| `kata-containers` | 4 | Static tarball → `/opt/kata`. KVM modules. k3s containerd handler `kata`. RuntimeClass `kata` with `node-tier=high-memory`. Needs `/dev/kvm` (nested virt if the worker is a VM). |
| `k3s-node-labeling` | 5 | `node-tier=high-memory\|mid-memory\|low-memory`. Taint masters `node-role.kubernetes.io/master=:NoSchedule`. |
| `openbao-setup` | 5 | Runs before ArgoCD exists. `helm install/upgrade` the `openbao/openbao` chart (values fetched from `k3s-manifests` `infrastructure/openbao/values.yaml`, the single source of truth). Generates OpenBao's own self-signed TLS Secret (`openbao-server-tls`) with `openssl` — no cert-manager yet. Waits for every `openbao-N` pod to reach `Running` (not `Ready` — that needs unsealing, which is the next role's job). |
| `openbao-secrets-init` | 5 | Refresh OpenBao EndpointSlices and probe every non-terminating health candidate from its local inventory node, preferring health 200/429 over sealed or uninitialized responses. Health and active discovery share one 600-second deadline; Pod probes time out after three seconds. Need `OPENBAO_BOOTSTRAP_TOKEN`. Re-discover until a serving, non-terminating active endpoint returns 200 before enabling KV v2, file audit, Kubernetes auth, and policy `external-secrets-read`. Bind SA `external-secrets-openbao`. Assert all seeded secrets are non-empty and meet length/PEM requirements, including `authentik_admin_token` (>= 32 chars). Seed all paths except `valkey`/`garage`/`platform-postgresql`'s `ca-cert` field (`tags: openbao, bootstrap`) — that field needs cert-manager's CA, which doesn't exist yet at this point; `openbao-ca-secrets` seeds it later. `end_role` if not ready. |
| `argocd-setup` | 5 | Official install.yaml into `argocd`. Pin Deployments/StatefulSet to `node-tier=high-memory`. Wait ready. Read initial admin secret. |
| `argocd-bootstrap` | 5 | Wait `applications.argoproj.io` CRD. Render `root-app.yaml.j2`. Apply. Watches `k3s-manifests` `infrastructure/` and `applications/` — OpenBao is not one of them. prune + selfHeal. |
| `openbao-ca-secrets` | 5 | Runs after `argocd-bootstrap`, once cert-manager has had a chance to sync. Re-discovers OpenBao's active endpoint (reuses `openbao-secrets-init`'s `discover-active-endpoint.yml`), fetches `selfsigned-ca-secret` from `cert-manager`, and `PATCH`es just the `ca-cert` field into `valkey`/`garage`/`platform-postgresql` without touching the rest of those KV entries. |
| `k9s-setup` | 5 | Install k9s. |
| `local-k9s-setup` | 6 | Fetch kubeconfig and configure k9s on the control node. |

## Part Talk to Part How

| From | To | How |
|---|---|---|
| Play 1 → play 2 | node-token | `hostvars[groups['masters'][0]]['k3s_node_token']['content']` base64-decoded. Workers never read the file. |
| Master → workers | API | `https://{{ k3s_master1_public_ip }}:6443`. Must be set. |
| Play 4 → worker | Kata | `/opt/kata` + k3s containerd tmpl. RuntimeClass via `masters[0]` kubeconfig. |
| Play 5 → cluster | kubeconfig | `/etc/rancher/k3s/k3s.yaml` via `kubernetes.core.k8s`. |
| `argocd-bootstrap` → Git | HTTPS | `https://github.com/freecloudinitiative/k3s-manifests.git` @ `HEAD`. |
| GitOps → OpenBao | cluster | Chart in `k3s-manifests`. Playbook does not install OpenBao. |
| `openbao-secrets-init` → OpenBao | HTTPS to the selected EndpointSlice Pod IP `:8200`, delegated to that Pod's inventory node | `X-Vault-Token` from env. `validate_certs: false` because dynamic Pod IPs are not certificate SANs. |
| External Secrets → OpenBao | later | Kubernetes auth. Role `external-secrets`. Audience `vault`. Token TTL 10m / max 30m. Playbook never stores bootstrap token in-cluster. |

## Why Build This Way

**Ansible for nodes, GitOps for apps — except OpenBao.** Node OS, k3s flags, join token, first Argo CD install need SSH and once-only ceremony. Everything after `root-app` must live in Git so drift heals — with one deliberate exception: OpenBao is entirely Ansible-owned (`openbao-setup` + `openbao-secrets-init`), never a GitOps `Application`, and runs *before* `argocd-bootstrap`. Reason: `external-secrets-config`'s `ClusterSecretStore` needs OpenBao's Kubernetes-auth backend configured before it can sync, so if OpenBao's readiness depended on ArgoCD (which is what happened when it was GitOps-managed), the very first sync would race a not-yet-configured OpenBao, fail validation, and every `ExternalSecret` in the cluster would sit unsynced until ArgoCD's automatic retry/selfHeal eventually caught up minutes later. Installing OpenBao first makes that race structurally impossible.

**Disable Traefik and ServiceLB at install.** Stock k3s ingress would fight GitOps Traefik and MetalLB.

**Token via hostvars, not a file on workers.** One slurp on primary. Workers never open `/var/lib/rancher/k3s/server/node-token`.

**OpenBao seed is a single-run operation, split across the boundary it can't avoid.** No dev root token. Operator init/unseal out of band. Env token, then revoke. Fail closed: no token → `end_role`, cluster still up. `authentik_admin_token` is not read from Authentik — it is an operator-chosen value seeded into OpenBao like every other secret and also passed to Authentik as `AUTHENTIK_BOOTSTRAP_TOKEN`, so Authentik creates its akadmin API token with that same value at boot (sync-wave 5). Both sides converge without a manual UI step or a second run. The one thing that genuinely can't happen before `argocd-bootstrap`: three secrets (`valkey`/`garage`/`platform-postgresql` `ca-cert`) need cert-manager's `selfsigned-ca-secret`, and cert-manager is still GitOps-managed. `openbao-ca-secrets` seeds just those three fields after `argocd-bootstrap` — still fully automatic in the same `ansible-playbook` run, not a manual second run.

**Labels and taints before Argo CD.** Argo CD pinned to `high-memory`. Masters and low-memory nodes do not take that load.

**Kata only on `high_memory`.** Needs KVM. RuntimeClass `kata` selects `node-tier=high-memory` so mid/low workers never get those pods. Restart `k3s-agent` happens before Argo CD is scheduled onto that node.

**No OpenBao token in Kubernetes.** External Secrets uses a ServiceAccount. Blast radius stays a 10-minute Vault token.

## Tool Use

| Tool | Why |
|---|---|
| Ansible 2.16+ | SSH + `become` on Ubuntu Jammy Pis. Roles, hostvars, vault. |
| Ansible Vault | `secret.yml` encrypted at rest. `ask_vault_pass`. |
| `kubernetes.core` | Apply Namespace, manifests, CRD wait, patches. |
| k3s | Single binary Kubernetes. `--cluster-init` HA-ready. Embedded registry for air-gapped-ish pulls. |
| Argo CD | App-of-apps. `root-app` is the only cluster object this repo keeps applying. |
| OpenBao | KV v2 for app secrets. Kubernetes auth for External Secrets. |
| Helm (on master) | Installed for operator use. App install is Argo CD, not `helm install` in this playbook. |

## Code Live Where When Run

- **Control node**: this repo. `ansible-playbook` from here. Needs Python 3, collections from `collections/requirements.yml`.
- **Masters / workers**: remote over SSH. `become: true`. k3s at `/usr/local/bin/k3s`. State under `/var/lib/rancher/k3s`. Kubeconfig `/etc/rancher/k3s/k3s.yaml` and `~/.kube/config`.
- **Argo CD**: namespace `argocd`. Manifest cached at `~/k3s-stack/argocd/install.yaml`. `root-app.yaml` rendered to `~/k3s-stack/argocd/root-app.yaml`.
- **OpenBao**: namespace `openbao` (GitOps). Seed discovers EndpointSlices and delegates each API call to the inventory node hosting that endpoint, preserving the namespace-only NetworkPolicy. Secrets under `secret/data/<path>`.
- **External Secrets**: namespace `external-secrets`. SA `external-secrets-openbao`. Role `external-secrets`.
