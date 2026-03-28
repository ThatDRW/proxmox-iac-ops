# proxmox-iac-ops

Enterprise-style Infrastructure-as-Code framework for managing **Proxmox VE** environments using **Ansible**.



## 🚀 Project Purpose
`proxmox-iac-ops` is a reusable automation framework designed to demonstrate **professional DevOps and Platform Engineering practices**.

The project simulates a real internal infrastructure repository used by an operations or platform team.

Core goals:
- Safe infrastructure automation
- Full lifecycle management
- Modular Ansible architecture
- Production-aware deployment safeguards
- Clean separation between public automation logic and private infrastructure data

This repository intentionally mirrors how real organizations structure infrastructure platforms.



## 🧱 Architecture Philosophy
### Safety First
Infrastructure changes must be:
- Reversible
- Predictable
- Observable

Every risky operation supports:
- Pre-change snapshots
- Verification checks
- Automatic rollback



### Full Lifecycle Management
Supported lifecycle operations:

`Create → Configure → Verify → Deploy → Rollback → Destroy`

Managed resources:
- Proxmox Virtual Machines (QEMU)
- Proxmox LXC Containers



### Separation of Concerns
#### Public Repository (this repo)
Contains:
- Ansible roles
- Playbooks
- Documentation
- Safe defaults
- Deployment logic

#### Private Repository (`infra-live`)
Contains:
- Inventory definitions
- Environment configuration
- Secrets (Ansible Vault)
- Real infrastructure state

No secrets are ever stored here.



### Environment Awareness
Infrastructure behavior changes based on environment:

```yaml
env: dev | staging | prod
```
Production safety guard:
```yaml
allow_prod_changes: true
```

Production changes require explicit operator intent.



## 📂 Repository Structure
```
proxmox-iac-ops/
├── docs/
├── playbooks/
├── roles/
│   ├── deployment_guard/
│   ├── pve_snapshot/
│   ├── pve_vm_lifecycle/
│   ├── pve_lxc_lifecycle/
│   ├── pve_rollback/
│   ├── os_base/
│   ├── os_hardening/
│   └── verify_common/
└── README.md
```



## ⚙️ Design Principles
### Modular Roles
Each role:
- Performs one responsibility
- Is independently reusable
- Remains idempotent

### Parameterized Infrastructure
Nothing is hardcoded.

Examples:
```
pve_api_host
pve_node
pve_vm_id
pve_lxc_id
```
All values originate from inventory.

### Safe Deployments
Typical deployment flow:
```
Deployment Guard
      ↓
Snapshot Creation
      ↓
Configuration Change
      ↓
Verification
      ↓
Success OR Rollback
```



## 🔒 Safety Model
Key protections:

- Automatic snapshots before change
- Block/rescue rollback handling
- Verification-gated deployments
- Production execution protection

See:
```
docs/safety_and_rollback.md
```



## 🧪 Verification System
Verification is role-based and extensible.

Examples:
- verify_ssh
- verify_network
- verify_service_<name>

Deployments succeed only when infrastructure is functionally healthy.



## 🐧 Supported Platforms
### Hypervisor
- Proxmox VE 9
### Operating Systems
- Debian 13 (Trixie)
- Alpine Linux



## 🔐 Secrets Management
Secrets are handled using:

- Ansible Vault
- Private infrastructure repository

_This project intentionally contains **no credentials**._



## 🧑‍💻 Intended Audience
This repository demonstrates capabilities relevant to:

- DevOps Engineers
- Platform Engineers
- Infrastructure Engineers
- Homelab automation enthusiasts



## 🛠️ Planned Features
- VM & LXC lifecycle automation
- Snapshot orchestration
- Automatic rollback workflows
- Verification framework
- OS baseline configuration
- Security hardening
- CI validation pipeline



## 📈 Development Approach
This project is built using incremental commits that simulate real-world engineering workflows:

- Small logical changes
- Clear commit history
- Design-first implementation
- Enterprise documentation standards



## 🚧 Project Status
Current Phase:

- [x] Foundation Documentation 
- [x] Core Ansible Framework 
- [x] Proxmox Integration 
- [ ] Deployment Flow 
- [ ] Verification System 
- [ ] CI/CD Integration



## 🤝 Contributing
This project is primarily a portfolio and learning initiative, but ideas and discussions are welcome.



## 📜 License
MIT License



## ⭐ Why This Project Exists
Most IaC examples show scripts.

This repository demonstrates a platform.

The goal is to build something indistinguishable from a real internal infrastructure automation framework used in production environments.



## 🚀 Deployment
Example deployment execution:

```bash
ansible-playbook playbooks/deploy.yml
```

Deployment workflow:
- Deployment guard validation
- Pre-change snapshot creation
- VM/LXC lifecycle execution
- Verification (future phase)