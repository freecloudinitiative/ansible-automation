# ROLES — How Code Work Together

## Big Picture

```
operator
  │
  │  ansible-playbook playbook.yml --ask-vault-pass
  ▼
playbook.yml
  │
  ├─ play 1  hosts: all
  │    inventory-validation (fail-fast, localhost only)
  │    raspberry-pi-cgroups → k3s-pre-setup
  │
  ├─ play 2  hosts: masters
  │    k3s-master-setup → k3s-fact-gathering
  │    first master: k3s server --cluster-init
  │    later masters: join :6443 (inventory has only master-1 uncommented)
  │    slurp node-token → hostvars
  │
  ├─ play 3  hosts: workers
  │    k3s-worker-setup
  │    token from hostvars[groups['masters'][0]]
  │
  ├─ play 4  hosts: high_memory
  │    kata-containers
  │    QEMU/KVM runtime on high-memory workers only
  │
  ├─ play 5  hosts: groups['masters'][0]
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
  │
  └─ play 6  hosts: groups['masters'][0]
       local-k9s-setup
       fetch kubeconfig, configure local k9s
```

Post-play debug prints public URLs. Passwords stay commented out.

## What Part Do

| Part | Job |
|---|---|
| `playbook.yml` | Six plays. Order is the contract. |
| `ssh-config.yml` | Nonprod SSH config. |
| `thermal-check.yml` | Check node temps. |
| `inventory.ini` | Host groups. `masters[0]` is primary. Memory groups drive labels/taints. |
| `group_vars/all/main.yml` | `k3s_*_public_ip` slots. Empty until filled. |
| `group_vars/all/secret.yml` | Vault. `--ask-vault-pass`. |
| `ansible.cfg` | Default inventory, vault prompt, `roles_path=./roles`. |
| `collections/requirements.yml` | `kubernetes.core` for k8s modules. |

## What Roles Do

| Role | Play | Job |
|---|---|---|
| `inventory-validation` | 1 | Validate required inventory variables (`k3s_master1_public_ip`, `masters`, `workers`) before modifying any nodes. Gated HA check when multiple masters exist. |
| `raspberry-pi-cgroups` | 1 | Memory cgroups enabled and asserted (`cgroup_memory=1 cgroup_enable=memory` in `cmdline.txt`; no-op on Ubuntu where already active). Fails play if controller still off after reboot. |
| `k3s-pre-setup` | 1 | Swap off. `dphys-swapfile` stopped. APT packages: wireguard, iscsi, nfs, containerd, python k8s libs. |
| `k3s-master-setup` | 2 | `get.k3s.io` server. `--disable traefik`, `--disable servicelb`, `--embedded-registry`. Readyz wait. Helm + CLI tools. |
| `k3s-fact-gathering` | 2 | Slurp node-token on `masters[0]`. Set `kubeconfig_path`. |
| `k3s-worker-setup` | 3 | `get.k3s.io` agent. `--server` + `--token`. `--node-name` from `worker_label`. |
| `kata-containers` | 4 | Static tarball → `/opt/kata`. KVM modules. k3s containerd handler `kata`. RuntimeClass `kata` with `node-tier=high-memory`. Needs `/dev/kvm` (nested virt if the worker is a VM). |
| `k3s-node-labeling` | 5 | `node-tier=high-memory\|mid-memory\|low-memory`. Taint low-memory `memory=limited:NoSchedule`. Taint masters `node-role.kubernetes.io/master=:NoSchedule`. |
| `argocd-setup` | 5 | Official install.yaml into `argocd`. Pin Deployments/StatefulSet to `node-tier=high-memory`. Wait ready. Read initial admin secret. |
| `argocd-bootstrap` | 5 | Wait `applications.argoproj.io` CRD. Render `root-app.yaml.j2`. Apply. Watches `k3s-manifests` `infrastructure/` and `applications/`. prune + selfHeal. |
| `openbao-secrets-init` | 5 | Wait for an OpenBao EndpointSlice and run the health check from its local inventory node. Health must be 200 or 429. Need `OPENBAO_BOOTSTRAP_TOKEN`. Wait for a serving, non-terminating active endpoint and verify it before enabling KV v2, file audit, Kubernetes auth, and policy `external-secrets-read`. Bind SA `external-secrets-openbao`. Assert all seeded secrets are non-empty and meet length/PEM requirements. Seed paths (`tags: openbao, bootstrap`). On first run, skips `authentik_admin_token` and prints a reminder. Run 2 (`--tags authentik-token`) PATCHes only `admin-token` into `secret/authentik` once the Authentik bootstrap admin exists. `end_role` if not ready. |
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

**Ansible for nodes, GitOps for apps.** Node OS, k3s flags, join token, first Argo CD install need SSH and once-only ceremony. Everything after `root-app` must live in Git so drift heals.

**Disable Traefik and ServiceLB at install.** Stock k3s ingress would fight GitOps Traefik and MetalLB.

**Token via hostvars, not a file on workers.** One slurp on primary. Workers never open `/var/lib/rancher/k3s/server/node-token`.

**OpenBao seed is a two-run operation.** No dev root token. Operator init/unseal out of band. Env token, then revoke. Fail closed: no token → `end_role`, cluster still up. Run 1 (`ansible-playbook playbook.yml`) seeds everything except `authentik_admin_token`, which does not exist until Authentik bootstraps at sync-wave 5. Run 2 (`--tags authentik-token`) PATCHes only that field once the token is created; no other secrets are overwritten. Until run 2 completes, iam-service starts normally but Authentik user sync fails silently.

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
