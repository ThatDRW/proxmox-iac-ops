# verify_common
Base verification framework for proxmox-iac-ops.



## Purpose
Provides a modular system for validating infrastructure health after deployment.

Verification determines deployment success and may trigger rollback.



## Variables
```
verification_enabled: true

verification_roles:
List of verification roles to execute.
```



## Example
```yaml
verification_roles:
  - verify_ssh
  - verify_network
```
Each verification role is independent and reusable.