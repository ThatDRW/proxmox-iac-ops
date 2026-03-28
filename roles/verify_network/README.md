# verify_network
Verifies network connectivity from within the managed host's own network stack.

Unlike `verify_ssh`, which runs from the controller, all checks here execute
**on the target host** — validating that the host itself has functional outbound
routing and DNS, not just that it's reachable from the outside.



## Checks
### 1. Default gateway reachability
Resolves the gateway from Ansible facts (`ansible_default_ipv4.gateway`) and
confirms it responds to ICMP ping. Fails immediately if no default gateway is
present in facts.

### 2. DNS resolution
Uses `getent hosts` to resolve a configurable hostname. Defaults to
`one.one.one.one` — a stable, well-known target that validates both the
resolver and upstream connectivity without depending on internal infrastructure.

### 3. External host reachability
TCP connectivity check against a configurable list of `host:port` pairs using
`wait_for`. Defaults to `1.1.1.1:443`. Override with internal endpoints for
air-gapped environments.



## Usage
```yaml
verification_roles:
  - verify_ssh
  - verify_network
```



## Variables
| Variable                              | Default           | Description                                     |
|---------------------------------------|-------------------|-------------------------------------------------|
| `verify_network_gateway_ping_count`   | `3`               | Number of ICMP pings to send                    |
| `verify_network_gateway_ping_timeout` | `5`               | Ping wait timeout in seconds                    |
| `verify_network_dns_hostname`         | `one.one.one.one` | Hostname to resolve for DNS check               |
| `verify_network_dns_timeout`          | `5`               | DNS resolution timeout in seconds               |
| `verify_network_external_hosts`       | `[1.1.1.1:443]`   | List of `{host, port, timeout}` to TCP-check    |

### External hosts format
```yaml
verify_network_external_hosts:
  - host: "1.1.1.1"
    port: 443
    timeout: 5
  - host: "internal-repo.example.com"
    port: 80
    timeout: 3
```

Set to `[]` to skip external reachability checks entirely.



## Notes
- Gateway is derived from `ansible_default_ipv4.gateway` — ensure `gather_facts: true`
  is set on the play (it is in `deploy.yml` by default)
- For Alpine hosts, `getent` is provided by `musl-utils` — included via `os_base`
