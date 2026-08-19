# FILES — Where Code Live

## Root

| File | What |
|---|---|
| `playbook.yml` | Main cluster playbook: k3s master, workers, labels, ArgoCD, OpenBao, k9s. |
| `pi-boot.yml` | Writes Raspberry Pi boot config (`/boot/firmware/config.txt`) on all nodes. |
| `local-setup.yml` | Fetches kubeconfig from master, sets up k9s on local machine. |
| `ssh-config.yml` | Generates local `~/.ssh/config` and populates `known_hosts`. |
| `thermal-check.yml` | Reads and prints Raspberry Pi CPU temperatures via `vcgencmd`. |
| `inventory.ini` | Ansible inventory: `[masters]`, `[workers]`, `[high_memory]`, `[mid_memory]`, `[low_memory]` groups. |
| `ansible.cfg` | Ansible config: default inventory, vault password prompt, SSH connection tuning. |
| `requirements.txt` | Python packages for the control machine (Ansible, etc.). |
| `caveman.md` | Documentation style guide for this repo. |
| `.ansible-lint` | ansible-lint config: rules, ignore patterns. |
| `.gitignore` | Excludes `.retry` files, temp dirs, local secrets. |

---

## collections/

| File | What |
|---|---|
| `requirements.yml` | Declares `kubernetes.core` and `community.general` Ansible collections. |

---

## group_vars/all/

| File | What |
|---|---|
| `main.yml` | Non-secret vars: public IPs for all nodes (`k3s_master1_public_ip`, etc.). |
| `secret.yml` | Ansible Vault–encrypted secrets: passwords, tokens, secret keys used by roles. |

---

## roles/raspberry-pi-boot-config/

| File | What |
|---|---|
| `tasks/main.yml` | Deploys boot config template; reboots Pi if changed. |
| `defaults/main.yml` | GPU memory split, cgroup flags, and other boot parameters. |
| `templates/config.txt.j2` | Jinja2 template for `/boot/firmware/config.txt`. |
| `README.md` | Role description. |

---

## roles/k3s-pre-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Disables swap, installs packages, installs Python k8s libraries. |
| `defaults/main.yml` | Package list (`k3s_presetup_packages`). |
| `meta/main.yml` | Role metadata (author, min Ansible version). |
| `README.md` | Role description. |

---

## roles/k3s-master-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Installs k3s server, Helm, kubectx, kubens, popeye, stern, kube-ps1. |
| `defaults/main.yml` | Tool versions (`kubectx_version`, `popeye_version`, `stern_version`, `kube_ps1_version`), kubeconfig path. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/k3s-worker-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Installs k3s agent, connects to primary master using node token. |
| `defaults/main.yml` | Default kubeconfig path. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/k3s-fact-gathering/

| File | What |
|---|---|
| `tasks/main.yml` | Reads node token from master disk; sets `kubeconfig_path` fact. |
| `defaults/main.yml` | Default kubeconfig path. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/k3s-node-labeling/

| File | What |
|---|---|
| `tasks/main.yml` | Labels nodes by memory tier; taints low-memory and master nodes. |
| `defaults/main.yml` | Default kubeconfig path. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/argocd-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Installs ArgoCD CLI + manifest, patches Deployments, waits for ready, reads admin password. |
| `defaults/main.yml` | ArgoCD version, namespace, manifest URL, node selector, deployment names. |
| `handlers/main.yml` | Handler: restarts `argocd-server` Deployment when ConfigMap changes. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/argocd-bootstrap/

| File | What |
|---|---|
| `tasks/main.yml` | Waits for Application CRD, renders and applies the GitOps root Application. |
| `defaults/main.yml` | GitOps repo URL, revision, bootstrap dir, `argocd_bootstrap_enabled` flag. |
| `templates/root-app.yaml.j2` | ArgoCD `Application` manifest (App of Apps). Points to `infrastructure/` and `applications/` paths in GitOps repo with auto-sync, prune, self-heal. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/openbao-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Adds Helm repo, creates namespace (restricted pod security), waits for CA issuer, creates TLS cert, deploys OpenBao via Helm, installs `bao` CLI. |
| `defaults/main.yml` | Namespace, Helm chart version, storage sizes, cert issuer name, CLI version, arch map. |
| `templates/openbao-values.yaml.j2` | Helm values for OpenBao: HA mode, TLS, storage, UI, audit log PVC. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/openbao-secrets-init/

| File | What |
|---|---|
| `tasks/main.yml` | Waits for OpenBao healthy, enables KV engine, configures Kubernetes auth, installs ESO policy, enables audit log, seeds all application secrets. |
| `defaults/main.yml` | Service names, namespace, bootstrap token var name, secret paths, audience, ESO role name. |
| `README.md` | Role description. |

---

## roles/k9s-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Downloads and installs k9s on master node, writes config and aliases for root. |
| `defaults/main.yml` | k9s version, install dir, arch map, kubeconfig path. |
| `templates/config.yaml.j2` | k9s `config.yaml` template (skin, default namespace, log settings). |
| `templates/aliases.yaml.j2` | k9s `aliases.yaml` template (command shortcuts). |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/local-k9s-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Fetches kubeconfig, patches master IP, installs k9s on local machine (Homebrew or direct download), templates k9s config locally. |
| `defaults/main.yml` | Local kubeconfig path, remote kubeconfig path, k9s version, `install_local_k9s` flag, `set_insecure_tls` flag. |
| `templates/config.yaml.j2` | k9s config template for local machine. |
| `templates/aliases.yaml.j2` | k9s aliases template for local machine. |
| `README.md` | Role description. |

---

## roles/ssh-config-setup/

| File | What |
|---|---|
| `tasks/main.yml` | Cleans known_hosts, writes `~/.ssh/config`, scans and adds host public keys. |
| `defaults/main.yml` | SSH config dir, config file path, known_hosts path, `ssh_config_clean_known_hosts` flag, `ssh_hosts` dict. |
| `templates/config.j2` | Jinja2 template for `~/.ssh/config`: one `Host` block per node with IP, user, identity file. |
| `meta/main.yml` | Role metadata. |
| `README.md` | Role description. |

---

## roles/rpi-thermal-check/

| File | What |
|---|---|
| `tasks/main.yml` | Runs `vcgencmd measure_temp`, prints hostname + temperature. Read-only. |
| `README.md` | Role description. |
