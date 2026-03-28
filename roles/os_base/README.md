# os_base
Provides baseline operating system configuration.



## Responsibilities
- Package updates
- Base tooling installation
- Python availability for Ansible execution
- OS abstraction



## Supported Systems
- Debian
- Alpine Linux



## Example
```yaml
- include_role:
    name: os_base
```
This role prepares hosts for higher-level configuration roles.