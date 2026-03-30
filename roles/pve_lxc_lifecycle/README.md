# pve_lxc_lifecycle
Manages lifecycle operations for Proxmox LXC containers.



## Supported Actions
pve_lxc_lifecycle_action:
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
pve_lxc_id
pve_lxc_lifecycle_template
```



## Destroy sequence
When `pve_lxc_lifecycle_action == "delete"` the following sequence runs:

1. Production guard — blocks destruction unless `pve_lxc_lifecycle_destroy_allow_prod=true`
2. Pre-destroy snapshot (optional) — last-chance restore point before permanent deletion
3. Stop the container — required before Proxmox will delete a guest
4. Delete the container (rootfs is always removed with the container)
5. Report outcome



## Destroy variables
| Variable                                   | Default  | Description                                               |
|--------------------------------------------|----------|-----------------------------------------------------------|
| `pve_lxc_lifecycle_destroy_stop_before`    | `true`   | Stop the container before deletion                        |
| `pve_lxc_lifecycle_destroy_stop_timeout`   | `60`     | Seconds to wait for the container to reach stopped state  |
| `pve_lxc_lifecycle_destroy_allow_prod`     | `false`  | Explicit opt-in required to destroy in production         |
| `pve_lxc_lifecycle_destroy_snapshot_before`| `true`   | Take a pre-destroy snapshot before deletion               |



## Example
```yaml
- include_role:
    name: pve_lxc_lifecycle
  vars:
    pve_lxc_lifecycle_action: create
```

To destroy in production:
```yaml
- include_role:
    name: pve_lxc_lifecycle
  vars:
    pve_lxc_lifecycle_action: delete
    pve_lxc_lifecycle_destroy_allow_prod: true
```

Handles hypervisor lifecycle only.
OS configuration is performed by separate roles.
