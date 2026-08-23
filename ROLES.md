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
  │    k3s-pre-setup → k3s-master-setup → k3s-fact-gathering
  │    first master: k3s server --cluster-init
  │    later masters: join :6443 (inventory has only master-1 uncommented)
  │    slurp node-token → hostvars
  │
  ├─ play 2  hosts: workers
  │    k3s-pre-setup → k3s-worker-setup
  │    token from hostvars[groups['masters'][0]]
  │
  └─ play 3  hosts: groups['masters'][0]
       k3s-node-labeling → argocd-setup → argocd-bootstrap
       → openbao-secrets-init
            │
            ▼
       Argo CD Application root-app
            │
            ▼
       k3s-manifests  (infrastructure/ + applications/)
            │
            ▼
       OpenBao (GitOps) ── wait, init/unseal, OPENBAO_BOOTSTRAP_TOKEN
            │
            ▼
       KV secrets + kubernetes auth + External Secrets policy
```

Post-play debug prints public URLs. Passwords stay commented out.

## What Part Do

| Part | Job |
|---|---|
| `playbook.yml` | Three plays. Order is the contract. |
| `inventory.ini` | Host groups. `masters[0]` is primary. Memory groups drive labels/taints. |
| `group_vars/all/main.yml` | `k3s_*_public_ip` slots. Empty until filled. |
| `group_vars/all/secret.yml` | Vault. `--ask-vault-pass`. |
| `ansible.cfg` | Default inventory, vault prompt, `roles_path=./roles`. |
| `collections/requirements.yml` | `kubernetes.core` for k8s modules. |

## What Roles Do

| Role | Play | Job |
|---|---|---|
| `k3s-pre-setup` | 1, 2 | Swap off. `dphys-swapfile` stopped. APT packages: wireguard, iscsi, nfs, containerd, python k8s libs. |
| `k3s-master-setup` | 1 | `get.k3s.io` server. `--disable traefik`, `--disable servicelb`, `--embedded-registry`. Readyz wait. Helm + CLI tools. |
| `k3s-fact-gathering` | 1 | Slurp node-token on `masters[0]`. Set `kubeconfig_path`. |
| `k3s-worker-setup` | 2 | `get.k3s.io` agent. `--server` + `--token`. `--node-name` from `worker_label`. |
| `k3s-node-labeling` | 3 | `node-tier=high-memory\|mid-memory\|low-memory`. Taint low-memory `memory=limited:NoSchedule`. Taint masters `node-role.kubernetes.io/master=:NoSchedule`. |
| `argocd-setup` | 3 | Official install.yaml into `argocd`. Pin Deployments/StatefulSet to `node-tier=high-memory`. Wait ready. Read initial admin secret. |
| `argocd-bootstrap` | 3 | Wait `applications.argoproj.io` CRD. Render `root-app.yaml.j2`. Apply. Watches `k3s-manifests` `infrastructure/` and `applications/`. prune + selfHeal. |
| `openbao-secrets-init` | 3 | Wait OpenBao Service. Health must be 200 or 429. Need `OPENBAO_BOOTSTRAP_TOKEN`. Enable KV v2, file audit, Kubernetes auth. Policy `external-secrets-read`. Bind SA `external-secrets-openbao`. Seed paths. Copy `selfsigned-ca-secret` `ca.crt` into Postgres/Valkey/Garage secrets. `end_role` if not ready. |

`openbao-setup` and `k9s-setup` sit in `roles/` but are commented out in `playbook.yml`. Not part of this path.

## Part Talk to Part How

| From | To | How |
|---|---|---|
| Play 1 → play 2 | node-token | `hostvars[groups['masters'][0]]['k3s_node_token']['content']` base64-decoded. Workers never read the file. |
| Master → workers | API | `https://{{ k3s_master1_public_ip }}:6443`. Must be set. |
| Play 3 → cluster | kubeconfig | `/etc/rancher/k3s/k3s.yaml` via `kubernetes.core.k8s`. |
| `argocd-bootstrap` → Git | HTTPS | `https://github.com/freecloudinitiative/k3s-manifests.git` @ `HEAD`. |
| GitOps → OpenBao | cluster | Chart in `k3s-manifests`. Playbook does not install OpenBao. |
| `openbao-secrets-init` → OpenBao | HTTPS to ClusterIP `:8200` | `X-Vault-Token` from env. `validate_certs: false` (private CA). |
| External Secrets → OpenBao | later | Kubernetes auth. Role `external-secrets`. Audience `vault`. Token TTL 10m / max 30m. Playbook never stores bootstrap token in-cluster. |

## Why Build This Way

**Ansible for nodes, GitOps for apps.** Node OS, k3s flags, join token, first Argo CD install need SSH and once-only ceremony. Everything after `root-app` must live in Git so drift heals.

**Disable Traefik and ServiceLB at install.** Stock k3s ingress would fight GitOps Traefik and MetalLB.

**Token via hostvars, not a file on workers.** One slurp on primary. Workers never open `/var/lib/rancher/k3s/server/node-token`.

**OpenBao seed is a second run.** No dev root token. Operator init/unseal out of band. Env token, then revoke. Fail closed: no token → `end_role`, cluster still up.

**Labels and taints before Argo CD.** Argo CD pinned to `high-memory`. Masters and low-memory nodes do not take that load.

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
- **OpenBao**: namespace `openbao` (GitOps). Seed talks to ClusterIP. Secrets under `secret/data/<path>`.
- **External Secrets**: namespace `external-secrets`. SA `external-secrets-openbao`. Role `external-secrets`.
