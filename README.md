# 🔐 VPN DevSecOps

> Automated deployment, security testing and continuous compliance validation of an **IPSec / TLS VPN**, built on DevSecOps principles.

![IaC](https://img.shields.io/badge/IaC-Terraform%20%7C%20Ansible-blueviolet)
![VPN](https://img.shields.io/badge/VPN-strongSwan%20%7C%20OpenVPN-blue)
![CI/CD](https://img.shields.io/badge/CI%2FCD-GitLab%20CI%20%7C%20GitHub%20Actions-orange)
![Security](https://img.shields.io/badge/Security-Checkov%20%7C%20OPA-green)

---

## 📖 Overview

This project applies **DevSecOps** principles to the deployment and hardening of a VPN (IPSec and/or TLS). The whole VPN infrastructure is described as code (**IaC**) and deployed through a **CI/CD pipeline**, which guarantees configurations that are **reproducible, traceable and secure**.

On every change, the pipeline:

- validates and scans the infrastructure code,
- enforces security policies (policy as code),
- deploys and configures the VPN,
- launches **automated attacks** against it (IKE brute force, downgrade attacks, DoS),
- verifies that **only authorized flows** cross the tunnel (ACL validation),
- **fails** if anything is non-compliant.

This is the **"Shift Left Security"** approach applied to network security.

## 🎯 What this project demonstrates

| Concept | Implementation |
|---|---|
| Infrastructure as Code | Terraform, Ansible |
| Security as Code | Automated hardening and VPN configuration |
| Policy as Code | Open Policy Agent (Rego policies) |
| Continuous Security Testing | ike-scan, Scapy, hping3, strongSwan test suite |
| Network Validation | Automated ACL / flow checks |
| Automated Attack Simulation | Downgrade, brute force, DoS |

## 🏗 Architecture

```
Git push
   ⬇
CI/CD (GitLab CI / GitHub Actions)
   ⬇
Terraform  →  network & firewall rules
   ⬇
Ansible    →  VPN install & configuration (IPSec / TLS)
   ⬇
VPN tunnel (IPSec / TLS)
   ⬇
Automated attack tests
   ⬇
Policy & ACL validation
   ⬇
Pipeline fails if non-compliant ❌ / passes ✅
```

**Lab environment**

- **VMware**: 2 Ubuntu VMs created manually (VPN server and VPN client)
- **Terraform**: manages network and firewall rules
- **Ansible**: installs and configures the VPN

## 🧰 Tech stack

| Category | Tools |
|---|---|
| Provisioning & network | Terraform |
| Configuration management | Ansible |
| VPN | strongSwan (IPSec / IKEv2), OpenVPN (TLS) |
| Attack & security testing | ike-scan, Scapy, hping3, strongSwan test suite |
| IaC & config validation | Checkov, Open Policy Agent (OPA) |
| CI/CD | GitLab CI or GitHub Actions |
| Monitoring | Docker-based monitoring stack |

## 🔄 CI/CD pipeline

| # | Stage | Tool | Goal |
|---|---|---|---|
| 1 | Validate | `terraform validate` | Syntax and structure of the IaC |
| 2 | IaC scan | Checkov | Detect insecure Terraform practices |
| 3 | Policy check | OPA | Allowed algorithms, IKE parameters, SA lifetime |
| 4 | Deploy infra | Terraform | Network and firewall rules |
| 5 | Configure VPN | Ansible | strongSwan / OpenVPN, ACLs, hardening |
| 6 | Attack tests | ike-scan, Scapy, hping3 | Brute force, downgrade, DoS |
| 7 | ACL validation | Automated tests | Only authorized flows cross the tunnel |
| 8 | Report | CI artifacts | Automatic test and compliance report |

## 🧪 Security tests

- **IKE discovery & brute force** (`ike-scan`): IKE version detection, configuration robustness, weak PSK resistance.
- **Downgrade attacks** (`Scapy`): forcing IKEv1 or weak crypto suites, crafted IKE packets, conformance checks.
- **DoS simulation** (`hping3`): behavior of the tunnel under flood or malformed traffic.
- **Cryptographic validation**: SAs use the expected algorithms; SPD/SAD only allow authorized flows.
- **strongSwan test suite**: integration and regression scenarios (NAT-T, multiple negotiations, etc.).

## ✅ Policy and compliance checks

- **Checkov** flags overly permissive ports and bad practices in Terraform.
- **OPA** blocks the pipeline when, for example, a non-approved cipher suite is used, an IKE parameter is invalid, or an SA lifetime is out of bounds.
- **ACL validation** tests confirm that only authorized flows pass through the tunnel and that everything else is blocked.

## 📁 Repository structure

```
vpn-devsecops/
├── ansible/
│   ├── inventories/
│   │   └── dev.ini
│   ├── roles/
│   │   ├── vpn-server/      # tasks, templates, handlers, vars
│   │   ├── vpn-client/
│   │   ├── firewall/
│   │   └── logging/
│   └── playbooks/
│       ├── vpn-server.yml
│       ├── vpn-client.yml
│       └── hardening.yml
├── docker/
│   └── monitoring/          # monitoring stack
├── scripts/                 # attack and test scripts
├── docs/                    # theoretical study and documentation
└── README.md
```

## 🚀 Getting started

### Prerequisites

- VMware (or equivalent) with **2 Ubuntu VMs** (server and client)
- Terraform, Ansible, Python 3 (Scapy), Docker
- `ike-scan`, `hping3`
- Checkov, OPA
- A GitLab or GitHub repository with CI/CD enabled

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/vpn-devsecops.git
cd vpn-devsecops

# 2. Provision network / firewall rules
cd terraform
terraform init
terraform validate
terraform apply

# 3. Configure the VPN
cd ../ansible
ansible-playbook -i inventories/dev.ini playbooks/vpn-server.yml
ansible-playbook -i inventories/dev.ini playbooks/vpn-client.yml
ansible-playbook -i inventories/dev.ini playbooks/hardening.yml
```

### Run the security checks locally

```bash
checkov -d terraform/                 # IaC scan
opa eval ...                          # policy check
ike-scan <vpn-server-ip>              # IKE discovery
python3 scripts/<downgrade_test>.py   # Scapy downgrade test
hping3 ...                            # DoS simulation
```

> ⚠️ Run attack tools **only** against your own lab environment.

## 📚 Documentation

Useful references:

- [IBM, What is DevSecOps](https://www.ibm.com/think/topics/devsecops)
- [strongSwan documentation](https://docs.strongswan.org/)
- [strongSwan test scenarios](https://strongswan.org/test-scenarios/)
- [Terraform](https://developer.hashicorp.com/terraform/intro/core-workflow) · [Ansible](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html)
- [Checkov](https://www.checkov.io/) · [OPA](https://www.openpolicyagent.org/docs)
- [Scapy](https://scapy.readthedocs.io/) · [ike-scan](https://www.kali.org/tools/ike-scan/) · [hping3](https://linux.die.net/man/8/hping3)

## 🗺 Roadmap

- [x] Terraform network and firewall modules
- [x] Ansible roles for strongSwan (IPSec)
- [x] OpenVPN (TLS) role
- [x] Automated attack scripts (Scapy, ike-scan, hping3)
- [x] OPA policies and Checkov integration
- [x] ACL validation
- [x] Full CI/CD pipeline with automatic report
- [x] Monitoring stack and anomaly detection

## ⚠️ Disclaimer

This project is for **educational purposes** (end-of-year project). All attack simulations must be run only in an isolated lab you own.

## 📄 License

Distributed under the MIT License. See [`LICENCE`](LICENCE) for more information.
