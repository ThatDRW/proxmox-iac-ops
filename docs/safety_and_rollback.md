# Safety and Rollback Model
## Overview
This document defines the safety mechanisms built into the proxmox-iac-ops framework.

The primary goal is to ensure that **all infrastructure changes are reversible, controlled, and predictable**, even in failure scenarios.



## Safety Principles
### 1. Default-Safe Operations
All potentially destructive or state-altering operations are protected by:

- Pre-change snapshots
- Conditional execution
- Environment safeguards

No operation should leave the system in an unrecoverable state.

### 2. Explicit Production Protection
Production environments are protected by default:

```yaml
env: prod
allow_prod_changes: false
```

To allow changes in production:

```yaml
allow_prod_changes: true
```

Destruction operations in production require an additional override:

```yaml
pve_vm_lifecycle_destroy_allow_prod: true
pve_lxc_lifecycle_destroy_allow_prod: true
```

### 3. Idempotency
All roles are:

- Idempotent
- Predictable
- Repeatable without side effects



## Snapshot Strategy
### Purpose
Snapshots provide a restore point before any risky operation.

### Default behaviour
```yaml
pve_snapshot_enabled: true
```

Snapshots are automatically taken before:
- Deployments (`pve_snapshot_operation: deploy`)
- Destroy operations (`pve_snapshot_operation: pre_destroy`)
- Any operation via manual invocation

### Naming convention
```
pre_<env>_<host>_<operation>_<epoch>
```

Example:

```
pre_staging_lxc-api-01_deploy_1743200000
```

Epoch is used over ISO8601 to stay within Proxmox's snapshot name length limit.

### Retention policy
Managed snapshots (prefixed `pre_`) are automatically pruned after creation:

```yaml
pve_snapshot_retention_enabled: true
pve_snapshot_retention_keep: 3
```

Manually created snapshots are never touched.

### Scope
Snapshots apply to:

- QEMU Virtual Machines
- LXC Containers



## Rollback Strategy
### Mechanism

Rollback is implemented using Ansible's `block/rescue` pattern in `deploy.yml`.
The rollback role stops the guest, restores the snapshot, and optionally restarts it.

### Rollback sequence
1. Assert `pve_rollback_snapshot` is defined
2. Stop the guest (required by Proxmox)
3. Restore the snapshot
4. Start the guest (configurable)

### Configuration
```yaml
pve_rollback_auto: true           # enable automatic rollback on failure
pve_rollback_stop_before: true    # stop guest before restoring (required)
pve_rollback_start_after: true    # start guest after successful restore
pve_rollback_stop_timeout: 60     # seconds to wait for clean shutdown
```

### Trigger conditions
Rollback is triggered when:

- A task in the `block` fails
- A verification step fails
- An assertion fails



## Verification-Gated Safety
Rollback is not only triggered by task failure, but also by **verification failure**.

```
Deploy → Verify → Fail → Rollback
```

This ensures functional correctness, not just successful task execution.

Verification roles run as part of the deployment block:

- `verify_ssh` - SSH port reachability and banner validation
- `verify_network` - gateway ping, DNS resolution, external reachability
- `verify_service` - service state via systemd or openrc



## Deployment Guard
`deployment_guard` enforces preconditions before any operation runs:

| Check | Behaviour |
|---|---|
| `env` not defined | Fails with descriptive message |
| `env` not in `dev/staging/prod` | Fails with descriptive message |
| `env == prod` and `allow_prod_changes != true` | Blocks execution |



## Failure Scenarios
| Scenario | Protection mechanism |
|---|---|
| Task failure | `block/rescue` rollback |
| Verification failure | Rollback triggered from rescue |
| Production misuse | `deployment_guard` + `allow_prod_changes` |
| Partial deployment | Snapshot restore |
| Rollback without snapshot | Assert fails with clear message |
| Destroy in production | `destroy_allow_prod` explicit opt-in |



## Recovery Model
### Automatic recovery
1. Snapshot taken before operation (`pve_snapshot_last_name` fact recorded)
2. Operation runs inside `block`
3. Failure occurs
4. `rescue` invokes `pve_rollback` with `pve_snapshot_last_name`
5. Guest restored and restarted

### Manual recovery
If `pve_rollback_auto` is disabled or a specific snapshot is needed:

```bash
ansible-playbook playbooks/deploy.yml \
  -i ../infra-live/inventories/staging \
  -l lxc-api-01 \
  --tags rollback \
  -e pve_rollback_snapshot=pre_staging_lxc-api-01_deploy_1743200000 \
  --ask-vault-pass
```



## Design Philosophy
Safety is enforced by design, not convention:

- Failures are expected and handled gracefully
- Recovery is fast, deterministic, and automated
- Production requires explicit intent at every level
