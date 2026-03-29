# verify_ssh
Verifies that SSH is reachable and responding correctly on managed hosts.

This role is designed to be used as a pluggable verification unit via the
`verify_common` dispatcher. It performs two sequential checks:

1. **TCP port reachability** — confirms the SSH port is open and accepting connections
2. **SSH banner validation** — retrieves the host's SSH banner via `ssh-keyscan`
   and asserts a valid protocol identifier is present

Both checks are performed from the Ansible controller (`delegate_to: localhost`),
making this role safe to run against newly provisioned VMs and LXC containers.



## Usage
Add this role to the `verify_common_roles` list in your playbook or group vars:

```yaml
verify_common_roles:
  - verify_ssh
```



## Variables
| Variable             | Default | Description                                    |
|----------------------|---------|------------------------------------------------|
| `verify_ssh_port`    | `22`    | SSH port to verify                             |
| `verify_ssh_timeout` | `10`    | Seconds to wait for port/banner response       |
| `verify_ssh_retries` | `3`     | Number of retries for port reachability check  |
| `verify_ssh_delay`   | `5`     | Seconds between retries                        |



## Integration
This role integrates with the `verify_common` dispatcher. Register it as part
of the verification framework to run it automatically during deployments:

```yaml
# group_vars/all.yml
verify_common_roles:
   - verify_ssh
```