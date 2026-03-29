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


## Example
```yaml
- include_role:
    name: pve_vm_lifecycle
  vars:
    pve_vm_lifecycle_action: create
```
This role manages hypervisor lifecycle only.
Guest configuration is handled separately.