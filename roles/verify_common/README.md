# verify_common
Base verification framework for proxmox-iac-ops.



## Purpose
Provides a modular system for validating infrastructure health after deployment.

Verification determines deployment success and may trigger rollback.



## Variables
```
verify_common_enabled: true

verify_common_roles:
List of verification roles to execute.
```



## Example
```yaml
verify_common_roles:
  - verify_ssh
  - verify_network
```
Each verification role is independent and reusable.