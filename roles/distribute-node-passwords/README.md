# distribute-node-password

Distributes a shared node password from Ansible Vault to all worker nodes securely.

## Description

This role deploys a pre-configured password (stored in Vault) to `/etc/rancher/node/password` on each worker node. The password is transferred securely via SSH key-based authentication and never exposed in logs or shell history.

**Key Features:**

- 🔐 Password sourced from Ansible Vault (encrypted)
- 🛡️ SSH key-based authentication (no plaintext over network)
- 📝 `no_log` prevents password exposure in logs
- ✅ Idempotent (safe to run multiple times)
- 🔍 Built-in security assertions

## Requirements

- Ansible 2.9+
- SSH key-based authentication configured
- `password` variable defined in `group_vars/all.yml` (encrypted with ansible-vault)

## Variables

| Variable   | Description                             | Required |
| ---------- | --------------------------------------- | -------- |
| `password` | The password to distribute (from Vault) | Yes      |

### Example: `group_vars/all.yml`

```yaml
---
password: "YourSecurePassword123!" # Encrypt with: ansible-vault encrypt group_vars/all.yml
```

Encrypt the file:

```bash
ansible-vault encrypt group_vars/all.yml
```

## Usage

### Inventory

```ini
[workers]
worker1.local
worker2.local
worker3.local
worker4.local
worker5.local
worker6.local
```

### Playbook

```yaml
---
- name: Configure worker nodes
  hosts: workers
  become: true
  roles:
    - distribute-node-password
```

### Run

```bash
# With vault password prompt
ansible-playbook -i inventory site.yml --ask-vault-pass

# With vault password file
ansible-playbook -i inventory site.yml --vault-password-file ~/.vault-pass
```

## Tasks

1. **Ensure rancher node directory exists** - Creates `/etc/rancher/node` with `0700` permissions
2. **Write node password from Vault** - Copies password from Vault to `/etc/rancher/node/password` with `0600` permissions
3. **Verify password file exists** - Stat check on the password file
4. **Assert password file is secure** - Validates file exists, has correct permissions (`0600`), and is owned by root

## File Structure

```
roles/distribute-node-password/
├── tasks/
│   └── main.yml
└── README.md
```

## Security Considerations

- **Vault Encryption**: Password is stored encrypted and decrypted only during playbook execution
- **No Log**: Password is never written to logs or Ansible output
- **SSH Key Auth**: Uses SSH keys for node authentication (not passwords)
- **File Permissions**: Password file is `0600` (readable only by root)
- **Ownership**: File is owned by root:root

## Example Output

```
TASK [distribute-node-password : Ensure rancher node directory exists] ****
changed: [worker1.local]
changed: [worker2.local]
changed: [worker3.local]

TASK [distribute-node-password : Write node password from Vault] ****
changed: [worker1.local]
changed: [worker2.local]
changed: [worker3.local]

TASK [distribute-node-password : Verify password file exists and has correct permissions] ****
ok: [worker1.local]
ok: [worker2.local]
ok: [worker3.local]

TASK [distribute-node-password : Assert password file is secure] ****
ok: [worker1.local]
ok: [worker2.local]
ok: [worker3.local]
```

## Idempotency

This role is fully idempotent. Running it multiple times with the same password will result in `changed=false` on subsequent runs:

```bash
$ ansible-playbook -i inventory site.yml --ask-vault-pass
# First run: changed=3
$ ansible-playbook -i inventory site.yml --ask-vault-pass
# Second run: changed=0 (no changes needed)
```

## Troubleshooting

### "Vault password is incorrect"

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass
# or
ansible-playbook -i inventory site.yml --vault-password-file ~/.vault-pass
```

### "Permission denied on /etc/rancher/node/password"

Ensure the task has `become: true` and node has sudo access without password for `mkdir`, `cp`, and `chmod`.

### "Assertion failed: Password file security check failed"

Check that `/etc/rancher/node/password` exists with correct permissions:

```bash
ssh worker1.local "ls -la /etc/rancher/node/password"
# Should show: -rw------- 1 root root
```
