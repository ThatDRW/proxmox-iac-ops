# pve_rollback
Restores Proxmox VMs or LXCs to a previous snapshot.



## Purpose
Provides automatic recovery after failed deployments. Typically executed
inside the `rescue` block of `deploy.yml` when a task or verification failure
is detected.



## Rollback sequence
For each host in scope:

1. Assert `pve_rollback_snapshot` is defined – fails fast with a clear message
   rather than passing an empty snapshot name to the API
2. Stop the guest (required by Proxmox before snapshot restore)
3. Restore the snapshot
4. Report the outcome
5. Start the guest (configurable)



## Variables
| Variable                    | Default                        | Description                                                    |
|-----------------------------|--------------------------------|----------------------------------------------------------------|
| `pve_rollback_auto`         | `true`                         | Toggle automatic rollback                                      |
| `pve_rollback_snapshot`     | `{{ pve_snapshot_last_name }}` | Snapshot to restore - defaults to the last managed snapshot    |
| `pve_rollback_stop_before`  | `true`                         | Stop the guest before rollback (required by Proxmox)           |
| `pve_rollback_stop_timeout` | `60`                           | Seconds to wait for the guest to reach stopped state           |
| `pve_rollback_start_after`  | `true`                         | Start the guest after a successful rollback                    |



## Example
```yaml
- ansible.builtin.include_role:
    name: pve_rollback
  vars:
    pve_rollback_snapshot: "{{ pve_snapshot_last_name }}"
```

Typically used inside a block/rescue deployment workflow:

```yaml
rescue:
  - ansible.builtin.include_role:
      name: pve_rollback
```
