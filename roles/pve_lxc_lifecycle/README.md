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



## Example
```yaml
- include_role:
    name: pve_lxc_lifecycle
  vars:
    pve_lxc_lifecycle_action: create
```
Handles hypervisor lifecycle only.
OS configuration is performed by separate roles.