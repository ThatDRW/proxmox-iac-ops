# Local Setup Guide
This guide covers everything required to run `proxmox-iac-ops` against a local Proxmox VE instance.
It assumes a working Proxmox VE node, a suitable control machine, and access to a paired
`infra-live` inventory repository.



## Prerequisites
### Control machine
The machine from which Ansible is executed. This can be a local workstation, a dedicated LXC
container on Proxmox, or a CI runner.

| Requirement       | Minimum version |
|-------------------|-----------------|
| Python            | 3.10            |
| Ansible           | 2.15            |
| ansible-lint      | 6.x             |
| Git               | any recent      |

> **Recommended:** Run Ansible from a dedicated LXC container on the Proxmox node itself.
> This eliminates network latency to the API and simplifies SSH routing to managed guests.

### Proxmox VE node
| Requirement          | Detail                                      |
|----------------------|---------------------------------------------|
| Proxmox VE           | 8.x or 9.x                                  |
| API access           | HTTPS on port 8006                          |
| API token            | Scoped to the relevant node and storage     |
| SSH access to guests | Control machine must reach managed guests   |



## Installation
### 1. Clone the repository
```bash
git clone https://github.com/your-org/proxmox-iac-ops.git
cd proxmox-iac-ops
```

Clone the private inventory repository alongside it:

```bash
git clone https://github.com/your-org/infra-live.git ../infra-live
```

The two repositories are expected to sit at the same directory level:

```
workspace/
├── proxmox-iac-ops/   ← this repo
└── infra-live/        ← private inventory
```

### 2. Install Python dependencies
```bash
pip install ansible ansible-lint
```

Or via a requirements file if your project defines one:

```bash
pip install -r requirements-dev.txt
```

### 3. Install Ansible collections
```bash
ansible-galaxy collection install -r requirements.yml -p ./collections
```

This installs `community.general` locally under `./collections`, as configured in `ansible.cfg`.



## Proxmox API Token
Authentication to the Proxmox API is performed exclusively via API tokens.
Passwords are not supported by this framework.

### Creating a token
1. Log in to the Proxmox web UI
2. Navigate to **Datacenter → Permissions → API Tokens**
3. Click **Add**
4. Select the user (e.g. `ansible@pve`)
5. Set a token ID (e.g. `automation`)
6. Uncheck **Privilege Separation** if the token should inherit the user's full permissions
7. Copy the secret - it is shown only once

### Minimum required permissions
Assign the following privileges to the token's ACL path (`/` or per-node):

| Privilege              | Purpose                            |
|------------------------|------------------------------------|
| `VM.Allocate`          | Create and destroy VMs and LXCs    |
| `VM.Config.Disk`       | Disk operations during create      |
| `VM.Config.CPU`        | CPU configuration                  |
| `VM.Config.Memory`     | Memory configuration               |
| `VM.Snapshot`          | Create and restore snapshots       |
| `VM.PowerMgmt`         | Start, stop, reboot                |
| `Datastore.AllocateSpace` | Write to storage                |
| `Sys.Audit`            | Read node and guest status         |



## Vault Configuration
API credentials are stored in Ansible Vault in the `infra-live` repository.
This repository contains no secrets.

### Creating the vault file
```bash
ansible-vault create ../infra-live/group_vars/all/vault.yml
```

Populate it with the following structure:

```yaml
# ../infra-live/group_vars/all/vault.yml  (encrypted)

pve_api_host: "192.168.1.10"
pve_api_user: "ansible@pve"
pve_api_token_id: "automation"
pve_api_token_secret: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

### Vault password file
For local use, store the vault password in a file that is never committed:

```bash
echo "your-vault-password" > ~/.vault_pass
chmod 600 ~/.vault_pass
```

Pass it at runtime:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/dev \
  --vault-password-file ~/.vault_pass
```

Or set it permanently in your shell environment:

```bash
export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass
```



## Inventory Setup
Inventory lives exclusively in `infra-live`. This repository includes only an example inventory
under `inventories/example/` for reference and syntax-check purposes.

A minimal dev inventory in `infra-live` follows this structure:

```
infra-live/
└── inventories/
    └── dev/
        ├── hosts.yml
        └── group_vars/
            └── all/
                ├── vars.yml
                └── vault.yml   ← encrypted
```

### Example `hosts.yml`
```yaml
all:
  children:
    pve_all:
      children:
        pve_vms:
          hosts:
            vm-web-01:
              ansible_host: 192.168.10.101

        pve_lxc:
          hosts:
            lxc-api-01:
              ansible_host: 192.168.10.201
```

### Example `group_vars/all/vars.yml`
```yaml
env: dev
pve_node: pve
pve_validate_certs: false

pve_snapshot_enabled: true
pve_snapshot_retention_enabled: true
pve_snapshot_retention_keep: 3

auto_rollback: true
allow_prod_changes: false
```

Host-specific variables such as `pve_vm_id` and `pve_lxc_id` belong in `host_vars/` per host.



## Verifying the Setup
### Syntax check
Validate the playbook and variable resolution without connecting to any host:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/dev \
  --syntax-check \
  --vault-password-file ~/.vault_pass
```

### Connectivity check
Verify that Ansible can reach managed guests over SSH:

```bash
ansible pve_all \
  -i ../infra-live/inventories/dev \
  -m ping \
  --vault-password-file ~/.vault_pass
```

### Dry run
Run the full playbook in check mode without applying any changes:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/dev \
  --check \
  --vault-password-file ~/.vault_pass
```

> Check mode will report `ok` or `changed` per task without modifying the system.
> Proxmox API calls that create or modify resources are skipped in check mode.

### Linting
Run the full lint suite before committing:

```bash
ansible-lint roles/ playbooks/
```



## SSH Access to Guests
Ansible connects to managed guests directly over SSH using the `ansible_host` defined in inventory.

The control machine must be able to reach each guest on port 22.
If guests are on an isolated VLAN, ensure the control machine has a routed path or is placed on the same network.

### SSH key setup
Generate a key pair for the Ansible control user if one does not already exist:

```bash
ssh-keygen -t ed25519 -C "ansible-control" -f ~/.ssh/ansible_id
```

Distribute the public key to all managed guests. For newly provisioned guests this can be done
via Proxmox cloud-init configuration or via a bootstrap task in the `os_base` role.

Set `ansible_ssh_private_key_file` in inventory or `group_vars` if using a non-default key:

```yaml
ansible_user: ansible
ansible_ssh_private_key_file: ~/.ssh/ansible_id
```



## Directory Reference
```
proxmox-iac-ops/
├── ansible.cfg              # Control machine config, collections path, forks
├── requirements.yml         # community.general collection
├── playbooks/
│   ├── deploy.yml           # Main deployment playbook
│   └── site.yml             # Full site playbook
├── roles/                   # All reusable roles
├── group_vars/              # Safe defaults (no secrets)
├── inventories/
│   └── example/             # Reference inventory (not for production use)
├── docs/                    # This and related documentation
└── collections/             # Installed by ansible-galaxy (not committed)
```



## Common Issues
### `community.general` module not found
Collections are installed locally under `./collections`. Ensure the install step was run from
the repository root so `ansible.cfg` can locate them:

```bash
ansible-galaxy collection install -r requirements.yml -p ./collections
```

### API token authentication failure
Confirm that `pve_api_token_id` is in the format `user@realm!token_id`, not just `token_id`.
The Proxmox API expects the full qualified form:

```yaml
pve_api_user: "ansible@pve"
pve_api_token_id: "automation"
```

These are passed to the `proxmox_kvm` and `proxmox` modules as separate parameters.

### SSH host key verification failure
`host_key_checking` is disabled in `ansible.cfg` by default for new guest provisioning.
If your security policy requires it, remove that setting and pre-populate `~/.ssh/known_hosts`
for all managed hosts before running playbooks.

### Snapshot operation fails on LXC
Proxmox requires the LXC container to be stopped before a snapshot can be restored.
The `pve_rollback` role handles this automatically when `pve_rollback_stop_before: true`.
If triggering a manual snapshot restore, ensure the container is not running.