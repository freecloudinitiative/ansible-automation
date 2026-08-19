# ROLES — What Roles Do

---

## raspberry-pi-boot-config

**Used by**: `pi-boot.yml` (all nodes)

Writes `/boot/firmware/config.txt` from a Jinja2 template. If file changes, reboots the Pi and waits 300 s for it to come back.

Sets GPU memory, cgroup memory/CPU flags required by Kubernetes, and any other boot-level hardware parameters.

**Run first, before any k3s work.** Reboot must complete before next playbook touches the node.

---

## k3s-pre-setup

**Used by**: `playbook.yml` (masters + workers)

Prepares every node before k3s installs:

1. Disables swap (`swapoff -a`, comments out fstab entry, stops `dphys-swapfile` service if present). K3s requires swap off.
2. Updates apt cache.
3. Installs required packages (e.g. `containerd`).
4. Ensures `containerd` is started and enabled.
5. Installs `python3-pip`, `python3-jsonpatch`, `python3-yaml`.
6. Upgrades `kubernetes>=24.2.0`, `jsonpatch`, `PyYAML` Python packages — required by `kubernetes.core` Ansible collection tasks later.

---

## k3s-master-setup

**Used by**: `playbook.yml` (masters group)

Installs and configures the k3s server on master nodes:

1. Downloads `https://get.k3s.io` install script.
2. **Primary master** (`masters[0]`): installs k3s with `--cluster-init`, disables built-in Traefik and ServiceLB (GitOps manages these instead), enables embedded registry.
3. **Secondary masters**: joins cluster using primary master IP + node token.
4. Waits for `/readyz` to return `ok` (retries 12×, 5 s delay).
5. Creates `/var/lib/rancher/k3s/storage` local storage directory.
6. Copies kubeconfig to user `~/.kube/config`, sets `KUBECONFIG` in `.bashrc`.
7. Installs **Helm** if not present.
8. Installs **kubectx** + **kubens** (arch-aware: arm64/amd64).
9. Installs **popeye** (Kubernetes cluster sanitizer).
10. Installs **stern** (multi-pod log tailer).
11. Installs **kube-ps1** (Kubernetes context in bash prompt), sources it in `.bashrc`.

---

## k3s-worker-setup

**Used by**: `playbook.yml` (workers group)

Installs k3s agent on each worker node:

1. Downloads k3s install script.
2. Installs k3s as `agent` pointing to primary master (`https://<master-ip>:6443`) using node token read by `k3s-fact-gathering`.
3. Worker name comes from `worker_label` inventory variable.

Runs after master setup. Node token is passed as a play variable decoded from `hostvars`.

---

## k3s-fact-gathering

**Used by**: `playbook.yml` (masters group, after master setup)

Bridge between master setup and worker setup:

1. Reads `/var/lib/rancher/k3s/server/node-token` from primary master. Sets it as a registered fact so workers can read it via `hostvars`.
2. Sets `kubeconfig_path` fact (`/etc/rancher/k3s/k3s.yaml`) so downstream roles use a consistent variable.

---

## k3s-node-labeling

**Used by**: `playbook.yml` (primary master only, `run_once`)

Applies Kubernetes node labels and taints based on inventory groups:

| Inventory group | Label applied | Taint applied |
|---|---|---|
| `high_memory` | `node-tier=high-memory` | none |
| `mid_memory` | `node-tier=mid-memory` | none |
| `low_memory` | `node-tier=low-memory` | `memory=limited:NoSchedule` |
| `masters` | none | `node-role.kubernetes.io/master=:NoSchedule` |

Checks which nodes are actually in the cluster before labelling — safe to run when some nodes are not yet joined.

---

## argocd-setup

**Used by**: `playbook.yml` (primary master)

Installs ArgoCD into the cluster:

1. Downloads `argocd` CLI binary (arch-aware).
2. Creates `argocd` namespace.
3. Downloads official ArgoCD install manifest.
4. Applies manifest to cluster via `kubernetes.core.k8s`.
5. Patches all ArgoCD Deployments with `nodeSelector` so they run on the correct node tier.
6. Configures `argocd-cmd-params-cm` ConfigMap for subpath routing (`server.insecure: false`). Restarts `argocd-server` via handler.
7. Waits for all ArgoCD Deployments to have all replicas ready (retries 60×, 10 s delay).
8. Reads the initial admin password from `argocd-initial-admin-secret` and sets it as a fact for the post-task output.

---

## argocd-bootstrap

**Used by**: `playbook.yml` (primary master, after `argocd-setup`)

Connects ArgoCD to the GitOps repository using the App of Apps pattern:

1. Waits for the `applications.argoproj.io` CRD to be established (retries 24×, 5 s delay).
2. Creates the bootstrap stack directory on master.
3. Renders `root-app.yaml.j2` → `root-app.yaml`. The root Application watches two paths in the GitOps repo:
   - `infrastructure/`: all infra app manifests
   - `applications/`: all service app manifests
   Both with `automated.prune=true` and `automated.selfHeal=true`.
4. Applies the root Application to Kubernetes. ArgoCD picks it up and starts reconciling the full stack.

Controlled by `argocd_bootstrap_enabled` flag. Set to `false` to skip (e.g. first-time run before GitOps repo is ready).

---

## openbao-setup

**Used by**: `playbook.yml` (primary master, after `argocd-bootstrap`)

Deploys OpenBao (open-source HashiCorp Vault fork) via Helm with mTLS:

1. Creates OpenBao stack directory.
2. Adds OpenBao Helm repository.
3. Creates `openbao` namespace and labels it with `pod-security.kubernetes.io/enforce: restricted`.
4. **Waits for the private CA `ClusterIssuer`** (`ca-cluster-issuer`) to be ready. This issuer is created by GitOps (cert-manager), so this step blocks until GitOps finishes deploying cert-manager.
5. Creates an ECDSA-256 TLS certificate via cert-manager for all OpenBao internal DNS names.
6. Waits for the TLS Secret to be provisioned.
7. Renders `openbao-values.yaml.j2` and deploys OpenBao Helm chart (`wait: false` — sealed pods are not Ready by design).
8. Installs `bao` CLI binary on master.
9. Writes `/etc/profile.d/openbao.sh` with `VAULT_ADDR` and `BAO_ADDR` env vars.
10. Copies OpenBao CA certificate to master for local admin use.

**After this role finishes, OpenBao pods run but are uninitialized and sealed.** Manual ceremony required before `openbao-secrets-init` can continue.

---

## openbao-secrets-init

**Used by**: `playbook.yml` (primary master, after `openbao-setup`)

Initializes OpenBao configuration and seeds all application secrets:

**Requires**: OpenBao initialized + unsealed (do this manually via `kubectl port-forward`), and `OPENBAO_BOOTSTRAP_TOKEN` env var set to a short-lived admin token.

Steps:

1. Ensures `external-secrets` namespace exists.
2. Waits for OpenBao health service to appear.
3. Polls `/v1/sys/health` until OpenBao is accessible (accepts 200, 429, 501, 503 — all mean "reachable").
4. Stops early (prints instructions) if OpenBao is uninitialized (501) or sealed (503).
5. Stops early (prints instructions) if no bootstrap token supplied.
6. Validates all required secret values meet minimum length: `AUTHENTIK_SECRET_KEY` ≥ 50 chars, passwords ≥ 24 chars, etc.
7. Enables versioned KV engine at `secret/` path (v2).
8. Installs `external-secrets-read` ACL policy (read-only on `grafana`, `authentik`, `platform-postgresql`, `valkey` paths).
9. Enables Kubernetes authentication method.
10. Configures Kubernetes auth (`kubernetes_host: https://kubernetes.default.svc:443`).
11. Binds External Secrets Operator service account to `external-secrets-read` policy (10 m TTL, 30 m max).
12. Enables file audit log at `/openbao/audit/audit.log`.
13. Seeds all application secrets into OpenBao KV (`/v1/secret/data/<path>`). All `no_log: true`.

Bootstrap token should be revoked immediately after this role completes.

---

## k9s-setup

**Used by**: `playbook.yml` (primary master)

Installs k9s (Kubernetes TUI) on the master node:

1. Detects CPU architecture (arm64/amd64).
2. Downloads and installs k9s binary to `/usr/local/bin/`.
3. Ensures kubeconfig is readable (`0644`).
4. Writes `/etc/profile.d/k9s_kubeconfig.sh` with `KUBECONFIG` env var.
5. Copies kubeconfig to `/root/.kube/config`.
6. Creates `~/.config/k9s/`, `~/.config/k9s/clusters/`, `~/.local/state/k9s/screen-dumps/` directories for root.
7. Templates `config.yaml` and `aliases.yaml` into `/root/.config/k9s/`.

---

## local-k9s-setup

**Used by**: `local-setup.yml` (runs on local control node, not on cluster)

Fetches kubeconfig from master and sets up k9s on the local development machine:

1. Ensures local `~/.kube/` directory exists.
2. Fetches kubeconfig from primary master to local `~/.kube/config`.
3. Replaces `127.0.0.1` in kubeconfig with the master's public IP so remote access works.
4. Replaces `certificate-authority-data` with `insecure-skip-tls-verify: true` for public IP access (when `set_insecure_tls=true`).
5. Sets kubeconfig permissions to `0600`.
6. Installs k9s on macOS via **Homebrew** (preferred). Falls back to direct binary download if Homebrew fails.
7. Creates local k9s config directories.
8. Templates `config.yaml` and `aliases.yaml` into local `~/.config/k9s/`.
9. Prints connection instructions.

---

## ssh-config-setup

**Used by**: `ssh-config.yml` (runs on local control node)

Generates a local `~/.ssh/config` file and populates `known_hosts` for all cluster nodes:

1. Ensures `~/.ssh/` directory exists (`0700`) and `known_hosts` file exists (`0600`).
2. Removes stale host keys from `known_hosts` (by IP and hostname) using `ssh-keygen -R`.
3. Renders `~/.ssh/config` from `config.j2` template with all node aliases, IPs, and SSH key paths.
4. Scans host public keys via `ssh-keyscan` (ed25519, ecdsa, rsa) and appends them to `known_hosts`.

After this, `ssh master-1`, `ssh worker-1` etc. work without accepting host key prompts.

---

## rpi-thermal-check

**Used by**: `thermal-check.yml` (all nodes)

Read-only. Runs `vcgencmd measure_temp` on each Raspberry Pi and prints the result:

```
master-1: 52.1'C
worker-1: 48.7'C
```

No changes made. Safe to run at any time.
