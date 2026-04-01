# Architecture Overview

## Purpose
This project provides a reusable Infrastructure-as-Code (IaC) framework for managing workloads on Proxmox VE using Ansible.

It is designed to model real-world platform engineering practices, with a strong focus on:
- Safety
- Modularity
- Reusability
- Lifecycle management
- Environment awareness



## System Overview
The system operates as an orchestration layer on top of Proxmox VE, managing both virtual machines and LXC containers through the Proxmox API.

```mermaid
graph TD
    CI[CI/CD Pipeline\nJenkins + GitLab] --> ANSIBLE[Ansible Control Layer\nplaybooks · roles]
    ANSIBLE --> GUARD[deployment_guard\nenv validation · prod gate]
    ANSIBLE --> PVE[Proxmox API\ncommunity.general]
    ANSIBLE --> OS[OS Configuration\nos_base · os_hardening]
    ANSIBLE --> VERIFY[Verification Layer\nverify_common dispatcher]

    PVE --> VM[QEMU VMs]
    PVE --> LXC[LXC Containers]
    PVE --> SNAP[Snapshots]

    OS --> VM
    OS --> LXC

    VERIFY --> VM
    VERIFY --> LXC
```



## Deployment Flow
A deployment runs as a linear sequence with a built-in rescue path. Every risky operation is bracketed by a snapshot and a rollback handler.

```mermaid
flowchart TD
    START([ansible-playbook deploy.yml]) --> GUARD[deployment_guard\nvalidate env · block prod]
    GUARD --> SNAP[pve_snapshot\npre_env_host_deploy_epoch]
    SNAP --> BLOCK

    subgraph BLOCK [block - rollback protected]
        LIFECYCLE[pve_vm_lifecycle /\npve_lxc_lifecycle] --> OSBASE[os_base\npackages · tooling]
        OSBASE --> HARD[os_hardening\nSSH · firewall · updates]
        HARD --> SVC[service_common\ninstall · enable · start]
        SVC --> VERIF[verify_common\ndispatcher]
        VERIF --> SSH[verify_ssh]
        VERIF --> NET[verify_network]
        VERIF --> SERV[verify_service]
    end

    BLOCK --> SUCCESS([Deployment complete])

    BLOCK -- failure --> RESCUE
    subgraph RESCUE [rescue]
        ROLLBACK[pve_rollback\nstop · restore · start]
    end
    RESCUE --> FAIL([Deployment failed\nrollback executed])
```



## Role Map
Roles are organised into five functional layers. Arrows indicate composition - a role at the head includes or depends on the role at the tail.

```mermaid
graph LR
    subgraph SAFETY [Safety]
        DG[deployment_guard]
        PS[pve_snapshot]
        PR[pve_rollback]
    end

    subgraph LIFECYCLE [Lifecycle]
        VM[pve_vm_lifecycle]
        LXC[pve_lxc_lifecycle]
    end

    subgraph OS [OS Configuration]
        OH[os_hardening]
        OB[os_base]
    end

    subgraph SERVICES [Services]
        SC[service_common]
        SNG[systemd nginx]
        SPG[openrc postgresql]
    end

    subgraph VERIFICATION [Verification]
        VC[verify_common]
        VS[verify_ssh]
        VN[verify_network]
        VSV[verify_service]
    end

    PS --> VM
    PS --> LXC
    PR --> VM
    PR --> LXC
    OB --> OH
    VC --> VS
    VC --> VN
    VC --> VSV
    VSV --> SC
    SC --> SNG
    SC --> SPG
```



## Repository Model
Logic and secrets are separated across two repositories. This repository contains everything that is safe to be public. Infrastructure-specific data lives in a private repository that is never published.

```mermaid
graph TD
    subgraph PUBLIC [proxmox-iac-ops - public]
        ROLES[roles/]
        PLAYS[playbooks/]
        DEFAULTS[safe defaults\ngroup_vars/]
        DOCS[docs/]
        CI[Jenkinsfile\n.ansible-lint]
    end

    subgraph PRIVATE [infra-live - private]
        INV[inventories/\ndev · staging · prod]
        VAULT[ansible-vault secrets\nAPI tokens · credentials]
        HVARS[host_vars/\npve_vm_id · pve_lxc_id]
        WRAP[wrapper playbooks]
    end

    PUBLIC -->|consumed by| PRIVATE
    VAULT -->|overrides| DEFAULTS
    INV -->|targets| ROLES
```



## Environment Model
Three environments are supported. Each inherits from the one above it in the hierarchy and overrides only what differs.

```mermaid
graph TD
    GVA[group_vars/all.yml\nenv=dev · safe defaults] --> GVP[group_vars/pve_all.yml\nAPI connection · node · timeout]
    GVP --> GVM[group_vars/pve_vms.yml\nVM lifecycle defaults]
    GVP --> GVL[group_vars/pve_lxc.yml\nLXC lifecycle defaults]

    GVM --> HV[host_vars per host\npve_vm_id · pve_lxc_id\ninfra-live only]
    GVL --> HV

    ENV{env} -->|dev| DEV[relaxed safety\nauto_rollback optional]
    ENV -->|staging| STG[full safety\nmirrors prod behaviour]
    ENV -->|prod| PRD[allow_prod_changes=true\nrequired for all changes]
```



## Authentication Model
Authentication to Proxmox is performed exclusively via API tokens. Credentials are stored in Ansible Vault in the private repository and are never present in this repository.

```mermaid
sequenceDiagram
    participant J as Jenkins
    participant A as Ansible LXC
    participant V as Ansible Vault
    participant P as Proxmox API

    J->>A: SCP playbooks + roles
    J->>A: SSH trigger ansible-playbook
    A->>V: Decrypt vault secrets
    V-->>A: pve_api_token_id / secret
    A->>P: API Token auth
    P-->>A: 200 OK
    A->>P: proxmox_kvm / proxmox module calls
    P-->>A: Task results
    A-->>J: Exit code · output
```



## Operating System Support
OS differences are handled via conditional includes within roles. No role assumes a specific init system or package manager.

| Concern             | Debian 13                 | Alpine Linux            |
|---------------------|---------------------------|-------------------------|
| Package manager     | `apt`                     | `community.general.apk` |
| Init system         | systemd                   | openrc                  |
| Service check       | `ansible.builtin.systemd` | `rc-service status`     |
| Firewall            | ufw                       | iptables                |
| Auto updates        | unattended-upgrades       | -                       |



## Design Goals
- Clarity over cleverness
- Safety over speed
- Modularity over monoliths

Intentionally designed to reflect real-world infrastructure engineering practices rather than minimal examples.
