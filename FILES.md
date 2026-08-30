# FILES — Where Code Live

Playbook path only. One line per file.

IGNORE `.github/` and `pi-boot.yml`.

---

## Root

| File | What |
|---|---|
| `playbook.yml` | Six plays: all hosts pre-setup, masters, workers, high_memory Kata, first-master components, local k9s. |
| `ssh-config.yml` | Nonprod SSH config. |
| `thermal-check.yml` | Check node temps. |
| `ansible.cfg` | `inventory.ini`, vault prompt, `roles_path=./roles`, SSH ControlMaster. |
| `inventory.ini` | Groups `masters`, `workers`, `high_memory`, `mid_memory`, `low_memory`. |
| `collections/requirements.yml` | `kubernetes.core`, `community.general`, `ansible.posix`, `community.grafana`. |
| `requirements.txt` | `ansible` pip pin for the control node. |
| `.ansible-lint` | Lint skip/warn. Excludes `.github`. |
| `.gitignore` | Retry files, venv, editor junk. |
| `caveman.md` | Doc style. |
| `README.md` | What `playbook.yml` does. |
| `ROLES.md` | How plays and roles connect. |
| `FILES.md` | This index. |

---

## group_vars/all/

| File | What |
|---|---|
| `group_vars/all/main.yml` | `k3s_*_public_ip` placeholders. Join address source. |
| `group_vars/all/secret.yml` | Ansible Vault ciphertext. Unlock with `--ask-vault-pass`. |

## roles/inventory-validation/

Validate required inventory variables before any node modification.

| File | What |
|---|---|
| `roles/inventory-validation/tasks/main.yml` | Fail-fast assertions for `k3s_master1_public_ip`, `masters`, and `workers` groups. |

---

## roles/k3s-pre-setup/

Runs on every master and worker before k3s.

| File | What |
|---|---|
| `roles/k3s-pre-setup/tasks/main.yml` | Swap off, fstab comment, stop `dphys-swapfile`, apt, containerd, pip k8s libs. |
| `roles/k3s-pre-setup/defaults/main.yml` | `k3s_presetup_packages` list. |
| `roles/k3s-pre-setup/meta/main.yml` | Galaxy metadata. |
| `roles/k3s-pre-setup/README.md` | Role notes. |

---

## roles/k3s-master-setup/

| File | What |
|---|---|
| `roles/k3s-master-setup/tasks/main.yml` | Download `get.k3s.io`. Primary `--cluster-init`. Secondary join. Readyz. Helm + kubectx/kubens/popeye/stern/kube-ps1. |
| `roles/k3s-master-setup/defaults/main.yml` | `kubeconfig_path`, tool versions. |
| `roles/k3s-master-setup/meta/main.yml` | Galaxy metadata. |
| `roles/k3s-master-setup/README.md` | Role notes. |

---

## roles/k3s-fact-gathering/

| File | What |
|---|---|
| `roles/k3s-fact-gathering/tasks/main.yml` | Slurp node-token on `masters[0]`. Set `kubeconfig_path`. |
| `roles/k3s-fact-gathering/defaults/main.yml` | Default kubeconfig path. |
| `roles/k3s-fact-gathering/meta/main.yml` | Galaxy metadata. |
| `roles/k3s-fact-gathering/README.md` | Role notes. |

---

## roles/k3s-worker-setup/

| File | What |
|---|---|
| `roles/k3s-worker-setup/tasks/main.yml` | `k3s` agent join using `k3s_master1_public_ip` + token. |
| `roles/k3s-worker-setup/defaults/main.yml` | `k3s_embedded_registry: true`. |
| `roles/k3s-worker-setup/meta/main.yml` | Galaxy metadata. |
| `roles/k3s-worker-setup/README.md` | Role notes. |

---

## roles/kata-containers/

Installs Kata Containers on `high_memory` workers. Needs `/dev/kvm`.

| File | What |
|---|---|
| `roles/kata-containers/tasks/main.yml` | KVM, static tarball → `/opt/kata`, k3s containerd handler, RuntimeClass `kata`. |
| `roles/kata-containers/defaults/main.yml` | `kata_version` `4.1.0`, guest memory/vCPU, RuntimeClass selector. |
| `roles/kata-containers/handlers/main.yml` | Restart `k3s-agent` or `k3s`. |
| `roles/kata-containers/templates/containerd-base.tmpl.j2` | k3s `{{ template "base" . }}` seed for missing containerd tmpls. |
| `roles/kata-containers/meta/main.yml` | Galaxy metadata. Needs `kubernetes.core`. |
| `roles/kata-containers/README.md` | Role notes. |

---

## roles/k3s-node-labeling/

| File | What |
|---|---|
| `roles/k3s-node-labeling/tasks/main.yml` | Label by inventory group. Taint low-memory and masters. |
| `roles/k3s-node-labeling/defaults/main.yml` | Label/taint maps (tasks use groups + kubectl, not this map). |
| `roles/k3s-node-labeling/meta/main.yml` | Galaxy metadata. |
| `roles/k3s-node-labeling/README.md` | Role notes. |

---

## roles/argocd-setup/

| File | What |
|---|---|
| `roles/argocd-setup/tasks/main.yml` | argocd CLI, namespace, install.yaml, nodeSelector patch, wait, admin secret. |
| `roles/argocd-setup/defaults/main.yml` | `argocd_version` `v3.5.1`, `argocd_deployments`, `node-tier: high-memory`. |
| `roles/argocd-setup/handlers/main.yml` | Restart `argocd-server` after cmd-params ConfigMap. |
| `roles/argocd-setup/meta/main.yml` | Galaxy metadata. |
| `roles/argocd-setup/README.md` | Role notes. |

---

## roles/argocd-bootstrap/

| File | What |
|---|---|
| `roles/argocd-bootstrap/tasks/main.yml` | Wait Application CRD. Render and apply `root-app`. |
| `roles/argocd-bootstrap/templates/root-app.yaml.j2` | App-of-apps. Paths `infrastructure` and `applications` in `k3s-manifests`. |
| `roles/argocd-bootstrap/defaults/main.yml` | Repo URL, `HEAD`, `argocd_bootstrap_enabled`. |
| `roles/argocd-bootstrap/meta/main.yml` | Galaxy metadata. Needs `kubernetes.core`. |
| `roles/argocd-bootstrap/README.md` | Role notes. |

---

## roles/openbao-secrets-init/

| File | What |
|---|---|
| `roles/openbao-secrets-init/tasks/main.yml` | Health gate, token gate, KV/auth/policy/audit, seed `openbao_secrets`. |
| `roles/openbao-secrets-init/defaults/main.yml` | Env lookups, namespace names, secret path list. |
| `roles/openbao-secrets-init/README.md` | Second-run env vars. Revoke token after. |

No `templates/`. No `meta/`.

---

## roles/k9s-setup/

| File | What |
|---|---|
| `roles/k9s-setup/tasks/main.yml` | Install k9s. |
