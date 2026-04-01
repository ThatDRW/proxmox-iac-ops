# Usage Examples
Practical reference for common operations against a configured inventory.
All commands assume you are in the repository root with a valid `infra-live`
inventory available and Ansible Vault unlocked manually.



## Running a full deployment
Deploy all hosts in the `pve_all` group against the staging inventory:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  --ask-vault-pass
```

Deploy against production (requires explicit override):

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/prod \
  --ask-vault-pass \
  -e allow_prod_changes=true
```



## Targeting specific hosts or groups
Limit execution to a single host:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  -l lxc-api-01 \
  --ask-vault-pass
```

Limit to all LXC containers:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  -l pve_lxc \
  --ask-vault-pass
```



## Using tags for selective execution
Run only OS hardening across all hosts:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  --tags hardening \
  --ask-vault-pass
```

Run only the verification framework:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  --tags verify \
  --ask-vault-pass
```

Skip snapshots and verification for fast iteration in dev:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/dev \
  --skip-tags snapshot,verify \
  --ask-vault-pass
```

Available tags: `guard`, `snapshot`, `lifecycle`, `os`, `hardening`, `services`, `verify`, `rollback`



## Syntax check (dry run)
Validate playbook structure and variable resolution without connecting to any host:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  --syntax-check
```



## Lifecycle operations
Create a new VM:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  -l vm-web-01 \
  -e pve_vm_lifecycle_action=create \
  --ask-vault-pass
```

Stop an LXC container:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  -l lxc-api-01 \
  --tags lifecycle \
  -e pve_lxc_lifecycle_action=stop \
  --ask-vault-pass
```

Destroy a VM in staging (pre-destroy snapshot taken automatically):

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  -l vm-web-01 \
  --tags lifecycle \
  -e pve_vm_lifecycle_action=delete \
  --ask-vault-pass
```

Destroy a VM in production (requires explicit override):

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/prod \
  -l vm-web-01 \
  --tags lifecycle \
  -e pve_vm_lifecycle_action=delete \
  -e pve_vm_lifecycle_destroy_allow_prod=true \
  --ask-vault-pass
```



## Manual snapshot
Take a snapshot outside of a deployment:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  --tags snapshot \
  -e pve_snapshot_operation=manual \
  --ask-vault-pass
```



## Manual rollback
Trigger a rollback to a specific snapshot:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  -l lxc-api-01 \
  --tags rollback \
  -e pve_rollback_snapshot=pre_staging_lxc-api-01_deploy_1743200000 \
  --ask-vault-pass
```



## Overriding safety defaults for dev iteration
Disable snapshots and auto rollback in dev to speed up iteration:

```yaml
# infra-live/group_vars/dev.yml
pve_snapshot_enabled: false
pve_rollback_auto: false
verification_enabled: false
```

Or pass inline:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/dev \
  -e pve_snapshot_enabled=false \
  -e pve_rollback_auto=false \
  --ask-vault-pass
```



## Ansible Vault
Encrypt a new secrets file:

```bash
ansible-vault encrypt ../infra-live/group_vars/all/vault.yml
```

Edit an existing encrypted file:

```bash
ansible-vault edit ../infra-live/group_vars/all/vault.yml
```

Run a playbook with a vault password file:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  --vault-password-file ~/.vault_pass
```



## Linting
Run the full lint suite locally before pushing:

```bash
ansible-lint roles/ playbooks/
```