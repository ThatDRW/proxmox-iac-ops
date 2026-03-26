# 🔒 Safety and Rollback Model

## 📌 Overview

This document defines the safety mechanisms built into the proxmox-iac-ops framework.

The primary goal is to ensure that **all infrastructure changes are reversible, controlled, and predictable**, even in failure scenarios.


## 🛡️ Safety Principles

### 1. Default-Safe Operations

All potentially destructive or state-altering operations are protected by:

- Pre-change snapshots
- Conditional execution
- Environment safeguards

No operation should leave the system in an unrecoverable state.



### 2. Explicit Production Protection

Production environments are protected by default.

```yaml
env: prod
allow_prod_changes: false
````

To allow changes:

```yaml
allow_prod_changes: true
```

This prevents accidental execution against critical infrastructure.



### 3. Idempotency

All roles must be:

* Idempotent
* Predictable
* Repeatable without side effects

This ensures safe retries and consistent outcomes.



## 📸 Snapshot Strategy

### Purpose

Snapshots provide a restore point before any risky operation.



### Default Behavior

```yaml
snapshot_enabled: true
```

Snapshots are automatically taken before:

* VM modifications
* LXC modifications
* Deployment actions



### Naming Convention

```
pre_<operation>_<timestamp>
```

Example:

```
pre_deploy_2026-03-24T20:15:00Z
```



### Scope

Snapshots apply to:

* QEMU Virtual Machines
* LXC Containers



## 🔁 Rollback Strategy

### Mechanism

Rollback is implemented using Ansible's:

```
block / rescue
```



### Example Pattern

```yaml
- block:
    - name: Perform risky operation
      include_role:
        name: some_role

  rescue:
    - name: Trigger rollback
      include_role:
        name: pve_rollback
```



### Trigger Conditions

Rollback is triggered when:

* A task fails
* A verification step fails
* A manual failure condition is raised



### Configuration

```yaml
auto_rollback: true
```

If disabled:

* Failures will stop execution
* No automatic rollback occurs



## 🧪 Verification-Gated Safety

Rollback is not only triggered by task failure, but also by **verification failure**.

Example flow:

```
Deploy → Verify → Fail → Rollback
```

This ensures:

* Functional correctness
* Not just successful execution



## 🚧 Deployment Guard

A dedicated role (`deployment_guard`) will enforce:

* Environment restrictions
* Required variables
* Safety checks before execution



## ⚠️ Failure Scenarios Covered

| Scenario           | Protection Mechanism      |
| ------------------ | ------------------------- |
| Task failure       | block/rescue rollback     |
| Misconfiguration   | Verification failure      |
| Production misuse  | Guard + explicit override |
| Partial deployment | Snapshot restore          |



## 🔄 Recovery Model

### Automatic Recovery

If enabled:

1. Snapshot is taken
2. Operation runs
3. Failure occurs
4. Rollback restores snapshot



### Manual Recovery

If auto rollback is disabled:

* Operator can manually invoke rollback role
* Snapshot naming ensures traceability



## 🧠 Design Philosophy

This system is designed to mimic **real-world platform engineering practices**, where:

* Safety is enforced by design, not convention
* Failures are expected and handled gracefully
* Recovery is fast, deterministic, and automated



## 🚀 Future Enhancements

* Snapshot retention policies
* Partial rollback strategies
* Change approval workflows
* Drift detection



## ✅ Summary

The safety model ensures:

* No irreversible changes
* Clear rollback paths
* Production-grade safeguards

This forms the foundation for all higher-level deployment logic in the project.