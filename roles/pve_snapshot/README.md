# pve_snapshot
Creates pre-change snapshots for Proxmox virtual machines and LXC containers.



## Purpose
Provides safety guarantees before infrastructure modifications.

Snapshots are typically executed before:
- Deployments
- Configuration changes
- Lifecycle operations



## Variables
```
snapshot_enabled: true

snapshot_operation:
    Logical operation name (deploy, update, lifecycle)
```



## Supported Targets
- QEMU Virtual Machines
- LXC Containers



## Example
```yaml
- include_role:
    name: pve_snapshot
  vars:
    snapshot_operation: deploy
```
Snapshots are skipped automatically when disabled.