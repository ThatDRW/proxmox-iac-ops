# os_hardening Role
Provides baseline operating system hardening.



## Scope (Stage 1)
- Package updates
- Automatic security updates
- SSH configuration hardening
- Basic firewall enablement



## Supported Systems
- Debian
- Alpine Linux



## Design Goals
- Safe defaults
- Idempotent
- Expandable toward CIS-style compliance

Future stages will introduce:

- auditd
- fail2ban
- kernel hardening
- logging policies



## Example
```yaml
- name: OS Hardening (Stage 1)
  include_role: 
    name: os_hardening
  tags: [hardening]
```