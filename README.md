# ansible-automation

## What Code Do

Ansible playbooks and roles that provision a bare-metal K3s Kubernetes cluster from scratch on Raspberry Pi nodes.

Run once → get a fully working cluster with:

- K3s installed on 1 master + 3 workers
- Nodes labelled by memory tier (`high-memory`, `mid-memory`, `low-memory`)
- ArgoCD running and connected to the GitOps repo (App of Apps pattern)
- OpenBao (open-source Vault) running with TLS, Kubernetes auth, and all application secrets seeded
- k9s TUI installed on master node and local machine
- Local `~/.kube/config` ready to use

Four playbooks, run in order:

| Playbook | What it does | Targets |
|---|---|---|
| `pi-boot.yml` | Writes `/boot/firmware/config.txt`. Reboots if changed. | all nodes |
| `playbook.yml` | Full cluster: k3s, labels, ArgoCD, OpenBao, k9s | masters + workers |
| `ssh-config.yml` | Writes local `~/.ssh/config`, scans known_hosts | localhost |
| `local-setup.yml` | Fetches kubeconfig, installs k9s on local machine | localhost |
| `thermal-check.yml` | Prints CPU temperature for all Raspberry Pi nodes | all nodes |

## Why Need It

No cloud provider. Bare-metal Raspberry Pi cluster needs manual provisioning. Ansible makes it idempotent and repeatable — run again after a node replacement or config change without breaking what already works.

## How Start

```bash
# 1. Install Ansible and dependencies
pip install -r requirements.txt

# 2. Install Ansible collections
ansible-galaxy collection install -r collections/requirements.yml

# 3. Fill in node IPs
# Edit group_vars/all/main.yml with real IPs for all nodes.
# Edit group_vars/all/secret.yml with secrets (encrypted with Ansible Vault).

# 4. Step 1: Write boot config to Raspberry Pi nodes and reboot
ansible-playbook pi-boot.yml

# 5. Step 2: Provision k3s cluster, ArgoCD, OpenBao, k9s
ansible-playbook playbook.yml

# IMPORTANT: OpenBao must be initialized and unsealed manually before
# step 5 completes. See openbao-secrets-init role section in ROLES.md.

# 6. (Optional) Set up local SSH config
ansible-playbook ssh-config.yml

# 7. Pull kubeconfig and set up local k9s
ansible-playbook local-setup.yml

# 8. (Anytime) Check Raspberry Pi temperatures
ansible-playbook thermal-check.yml
```

Vault password prompt appears automatically (`ask_vault_pass = true` in `ansible.cfg`).

## Language

YAML. Ansible 2.x. Python 3 required on target nodes (installed automatically). Collections: `kubernetes.core`, `community.general`.

## Folders

```
roles/                  One folder per role. Each has tasks/, defaults/, templates/, meta/.
  raspberry-pi-boot-config/   Write Pi boot config, reboot.
  k3s-pre-setup/              Disable swap, install packages, Python k8s libs.
  k3s-master-setup/           Install k3s server, Helm, kubectx, popeye, stern, kube-ps1.
  k3s-worker-setup/           Install k3s agent, connect to master.
  k3s-fact-gathering/         Read node token, set kubeconfig_path fact.
  k3s-node-labeling/          Label and taint nodes by memory tier.
  argocd-setup/               Install ArgoCD CLI and manifests, wait for ready.
  argocd-bootstrap/           Apply GitOps root Application (App of Apps).
  openbao-setup/              Deploy OpenBao via Helm with TLS from cert-manager.
  openbao-secrets-init/       Initialize KV engine, Kubernetes auth, seed secrets.
  k9s-setup/                  Install k9s on master node.
  local-k9s-setup/            Fetch kubeconfig, install k9s on local machine.
  ssh-config-setup/           Write ~/.ssh/config, populate known_hosts.
  rpi-thermal-check/          Print CPU temperature.
group_vars/all/
  main.yml                    Node IPs and non-secret vars.
  secret.yml                  Encrypted secrets (Ansible Vault).
collections/
  requirements.yml            Ansible collection dependencies.
```

## Read More

- [ROLES.md](ROLES.md) — what each role does, step by step
- [FILES.md](FILES.md) — every file, one line
