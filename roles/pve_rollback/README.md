# pve_rollback
Restores Proxmox VMs or LXCs to a previous snapshot.



## Purpose
Provides automatic recovery after failed deployments.



## Variables
```
auto_rollback: true
rollback_snapshot: snapshot name
```



## Example
```yaml
- include_role:
    name: pve_rollback
```

Typically executed inside a block/rescue deployment workflow.