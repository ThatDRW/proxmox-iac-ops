# pve_snapshot
Creates pre-change snapshots for Proxmox virtual machines and LXC containers,
and enforces a configurable retention policy after each snapshot is taken.

Snapshots are taken before risky operations and serve as the restore point
for the `pve_rollback` role. The snapshot name is stored as a host fact
(`pve_snapshot_last_name`) so it can be referenced by the rescue block in
`deploy.yml` without requiring manual coordination.



## Snapshot naming
Names are generated deterministically from runtime context:

```
pre_<env>_<host>_<operation>_<epoch>
```

Example: `pre_staging_lxc-api-01_deploy_1743200000`

Epoch is used instead of ISO8601 to stay within Proxmox's snapshot name
length limit and avoid characters that are invalid in snapshot identifiers.



## Metadata
A structured description is written to the Proxmox snapshot record:

```
env=staging host=lxc-api-01 operation=deploy timestamp=2025-01-01T12:00:00Z triggered_by=ansible
```

This makes snapshots auditable directly from the Proxmox UI or API without
needing to cross-reference Ansible logs.



## Retention policy
After each snapshot is created, the retention policy is evaluated automatically.

Only snapshots whose names begin with `pre_` are considered — snapshots created
manually via the Proxmox UI or API are never touched. Managed snapshots are
sorted by creation time and the oldest beyond `pve_snapshot_retention_keep`
are deleted via the Proxmox API.



## Variables
| Variable                          | Default                                 | Description                                        |
|-----------------------------------|-----------------------------------------|----------------------------------------------------|
| `pve_snapshot_enabled`            | `true`                                  | Toggle snapshot creation                           |
| `pve_snapshot_operation`          | `operation`                             | Logical operation label (e.g. `deploy`, `update`)  |
| `pve_snapshot_name`               | `pre_<env>_<host>_<operation>_<epoch>`  | Generated snapshot identifier                      |
| `pve_snapshot_metadata`           | see defaults                            | Dict of values written to snapshot description     |
| `pve_snapshot_description`        | rendered from `pve_snapshot_metadata`   | String written to Proxmox snapshot description     |
| `pve_snapshot_retention_enabled`  | `true`                                  | Toggle retention enforcement                       |
| `pve_snapshot_retention_keep`     | `3`                                     | Number of managed snapshots to retain per host     |



## Outputs
| Fact                      | Description                                              |
|---------------------------|----------------------------------------------------------|
| `pve_snapshot_last_name`  | The name of the snapshot just created — used by `pve_rollback` |



## Example
```yaml
- ansible.builtin.include_role:
    name: pve_snapshot
  vars:
    pve_snapshot_operation: deploy
```



## Supported targets
- QEMU Virtual Machines (`pve_vms` group)
- LXC Containers (`pve_lxc` group)