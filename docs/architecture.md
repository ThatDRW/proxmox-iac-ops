# Architecture Overview





## 📌 Purpose


This project provides a reusable Infrastructure-as-Code (IaC) framework for managing workloads on Proxmox VE using Ansible.

It is designed to model real-world platform engineering practices, with a strong focus on:
- Safety
- Modularity
- Reusability
- Lifecycle management
- Environment awareness




## 🧱 System Overview


The system operates as an orchestration layer on top of Proxmox VE, managing both virtual machines (VMs) and LXC containers through the Proxmox API.

High-level responsibilities include:
- Provisioning infrastructure
- Applying configuration
- Verifying system state
- Handling failures with rollback mechanisms




## 🧩 Core Components


### 1. Ansible Control Layer
- Executing playbooks
- Coordinating roles
- Managing state transitions

This layer acts as the **orchestrator** of all infrastructure operations.


### 2. Proxmox Interaction Layer

Handles communication with Proxmox VE via API.

Responsibilities:
- VM lifecycle management
- LXC lifecycle management
- Snapshot creation and rollback

All interactions are abstracted through dedicated roles using the `pve_` naming convention.



### 3. Workload Layer

Represents the managed systems:
- QEMU Virtual Machines
- LXC Containers

Each workload is treated as a managed node with:
- OS-level configuration
- Service-level configuration
- Verification hooks



### 4. Verification Layer

A modular system of verification roles used to validate system state after changes.

Examples:
- SSH availability
- Network reachability
- Service health

Verification is:
- Composable
- Reusable
- Extensible



### 5. Safety Layer

Implements safeguards to prevent unsafe operations.

Includes:
- Pre-change snapshots
- Environment-based restrictions
- Automatic rollback on failure




## 🔄 Execution Flow


A typical deployment follows this sequence:

```
  [Deployment Guard]
          ↓
  [Snapshot Creation]
          ↓
[Provision / Configure]
          ↓
    [Verification]
          ↓
      [Success] → End
```

OR

```
[Failure]
↓
[Rollback to Snapshot]
```





## 🧭 Environment Model


The system supports multiple environments:
- dev
- staging
- prod

Environment-specific behavior is controlled through variables.

Production environments enforce stricter safety controls, requiring explicit overrides for changes.





## 🔐 Authentication Model


Authentication to Proxmox is performed using API tokens.

This enables:
- Secure automation
- Separation of credentials from code
- Compatibility with secret management systems




## 🐧 Operating System Support


The framework supports multiple Linux distributions:
- Debian-based systems
- Alpine Linux systems

Differences between distributions are handled via conditional logic and abstraction within roles.





## 🧩 Role-Based Architecture


The system is structured around modular Ansible roles.

Each role has a single responsibility and can be reused independently.

Examples include:
- Lifecycle management roles (VM/LXC)
- Snapshot and rollback roles
- OS configuration roles
- Verification roles





## 🔄 Idempotency


All operations are designed to be idempotent:
- Running the same playbook multiple times should not produce unintended changes
- State is always converged toward the desired configuration





## 📦 Repository Separation


This project is designed to work alongside a separate private repository.

Public repository:
- Contains reusable roles and playbooks
- Contains documentation and safe defaults

Private repository:
- Contains inventory
- Contains secrets (encrypted)
- Contains environment-specific configuration





## 🎯 Design Goals


- Provide a realistic enterprise-style infrastructure framework
- Implementation of safe automation practices
- Enable full lifecycle management of workloads
- Maintain clarity and modularity at scale
- Reduce complexity and improve maintainability





## 🚀 Future Extensions


Potential future improvements include:
- Snapshot retention policies (automated cleanup)
- Advanced verification modules (e.g., monitoring, health checks, etc.)
- Integration with external secret management systems (HashiCorp Vault, Secrets Managers, etc.)
- CI/CD-driven deployment pipelines (Jenkins, GitLab, etc.)
- Support for k8s cluster orchestration





## 🧠 Summary



This architecture emphasizes:
- Clarity over cleverness
- Safety over speed
- Modularity over monoliths

It is intentionally designed to reflect real-world infrastructure engineering practices rather than minimal examples.