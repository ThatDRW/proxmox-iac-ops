# pve_vm_lifecycle
Manages lifecycle operations for Proxmox virtual machines.



## Supported Actions
pve_vm_lifecycle_action:
- create
- start
- stop
- delete



## Required Variables
```
pve_api_host
pve_api_user
pve_api_token_id
pve_api_token_secret
pve_node
pve_vm_id
```



## Destroy sequence
When `pve_vm_lifecycle_action == "delete"` the following sequence runs:

1. Production guard — blocks destruction unless `pve_vm_lifecycle_destroy_allow_prod=true`
2. Pre-destroy snapshot (optional) — last-chance restore point before permanent deletion
3. Stop the VM — required before Proxmox will delete a guest
4. Delete the VM
5. Report outcome



## Destroy variables
| Variable                                  | Default  | Description                                              |
|-------------------------------------------|----------|----------------------------------------------------------|
| `pve_vm_lifecycle_destroy_stop_before`    | `true`   | Stop the VM before deletion                              |
| `pve_vm_lifecycle_destroy_stop_timeout`   | `60`     | Seconds to wait for the VM to reach stopped state        |
| `pve_vm_lifecycle_destroy_allow_prod`     | `false`  | Explicit opt-in required to destroy in production        |
| `pve_vm_lifecycle_destroy_snapshot_before`| `true`   | Take a pre-destroy snapshot before deletion              |
| `pve_vm_lifecycle_destroy_purge_disks`    | `false`  | Purge associated disks from storage on deletion          |



## Example
```yaml
- include_role:
    name: pve_vm_lifecycle
  vars:
    pve_vm_lifecycle_action: create
```

To destroy in production:
```yaml
- include_role:
    name: pve_vm_lifecycle
  vars:
    pve_vm_lifecycle_action: delete
    pve_vm_lifecycle_destroy_allow_prod: true
```

This role manages hypervisor lifecycle only.
Guest configuration is handled separately.
