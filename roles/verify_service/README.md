# verify_service
Verifies that expected services are active on managed hosts.

Supports both **systemd** (Debian) and **openrc** (Alpine), consistent with the
OS abstraction strategy used across `os_base` and `os_hardening`.



## Service list resolution
The role resolves which services to check using the following precedence:

1. `verify_services` - if defined and non-empty, used exclusively
2. `services` - fallback to the list managed by `service_common`

This allows `verify_service` to be dropped into any play without additional
configuration when `service_common` is already in use, while still supporting
independent verification targets where needed.



## Usage
### Minimal - relies on service_common's list
```yaml
# group_vars/pve_all.yml
services:
  - name: nginx
  - name: fail2ban

verification_roles:
  - verify_ssh
  - verify_network
  - verify_service
```

### Explicit override
```yaml
# host_vars/lxc-api-01.yml
verify_services:
  - name: nginx
  - name: postgresql
```



## Variables
| Variable          | Default | Description                                              |
|-------------------|---------|----------------------------------------------------------|
| `verify_services` | `[]`    | Explicit list of services to verify. Falls back to `services` when empty. |

### Service item format
Matches the `service_common` convention:

```yaml
services:
  - name: nginx
  - name: fail2ban
```



## OS support
| OS family | Init system | Check command              |
|-----------|-------------|----------------------------|
| Debian    | systemd     | `systemctl is-active <svc>`|
| Alpine    | openrc      | `rc-service <svc> status`  |