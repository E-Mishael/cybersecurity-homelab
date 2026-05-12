# 08 — Roadmap

## Overview

This homelab is a living project. What exists today is the foundation — a stable, segmented, firewall-enforced virtual environment. What comes next transforms it into a full security operations simulation platform.

Each planned phase has a clear technical objective, a set of skills it develops, and a direct mapping to real-world security job functions.

---

## Current State (Completed)

| Component | Status | Documentation |
|:---|:---|:---|
| Proxmox VE hypervisor (bare metal) | ✅ Complete | [02-proxmox-setup.md](02-proxmox-setup.md) |
| Dual-bridge network segmentation | ✅ Complete | [03-network-architecture.md](03-network-architecture.md) |
| pfSense firewall (NAT, DHCP, rules) | ✅ Complete | [04-pfsense-firewall.md](04-pfsense-firewall.md) |
| Suricata IDS/IPS on pfSense | ✅ Complete | [05-suricata-ids.md](05-suricata-ids.md) |
| WireGuard VPN remote access | ✅ Complete | [06-wireguard-vpn.md](06-wireguard-vpn.md) |
| Ubuntu Server VM with Xubuntu GUI | ✅ Complete | [02-proxmox-setup.md](02-proxmox-setup.md) |
| Troubleshooting and documentation | ✅ Complete | [07-troubleshooting-log.md](07-troubleshooting-log.md) |

---

## Phase 1 — Wazuh SIEM Deployment *(In Progress)*

### Objective
Deploy Wazuh as a centralized Security Information and Event Management (SIEM) platform to collect, correlate, and alert on security events across all lab systems.

### What Will Be Built
- Wazuh Manager installed on Ubuntu Server VM
- Wazuh agents deployed on all lab VMs (Ubuntu, future Windows)
- Suricata EVE JSON logs forwarded to Wazuh
- pfSense firewall logs forwarded to Wazuh
- Custom alert rules for lab-specific detection scenarios
- Wazuh dashboard configured for real-time monitoring

### Skills Developed
- SIEM deployment and administration
- Log source onboarding and normalisation
- Alert rule writing and tuning
- Security event correlation
- Dashboard creation and monitoring workflows

### Career Mapping
Directly relevant to: SOC Analyst (Tier 1/2), Security Engineer, Detection Engineer roles

---

## Phase 2 — Kali Linux Attack Simulation

### Objective
Deploy a Kali Linux VM inside the lab network and conduct authorised attack simulations against other lab VMs — generating real security events for detection and response practice.

### What Will Be Built
- Kali Linux VM on vmbr1 (internal network)
- Structured attack scenarios documented as lab exercises:
  - Network reconnaissance (Nmap, Netdiscover)
  - Vulnerability scanning (Nessus, Nikto)
  - Exploitation attempts (Metasploit against intentionally vulnerable targets)
  - Password attacks (Hydra, John the Ripper)
  - Traffic capture and analysis (Wireshark, tcpdump)
- Each attack correlated with Suricata alerts and Wazuh events
- Formal write-up: attack → detection → response

### Skills Developed
- Offensive security methodology
- Attack pattern recognition
- Alert correlation (matching attack to SIEM event)
- Incident response workflow
- Penetration test report writing

### Career Mapping
Directly relevant to: Penetration Tester, Red Team Analyst, SOC Analyst, Security Engineer roles

---

## Phase 3 — Windows Endpoint and EDR Testing

### Objective
Add a Windows 10/11 VM to the lab to simulate a corporate endpoint environment, deploy endpoint detection tooling, and practice Windows-specific security monitoring.

### What Will Be Built
- Windows 10/11 VM on vmbr1
- Wazuh agent deployed on Windows — Windows Event Log forwarding
- Sysmon installed for enhanced Windows telemetry
- EDR testing against Kali attack simulation
- Active Directory (optional) — simulate domain environment
- Detection of common Windows-based attacks (Pass-the-Hash, LSASS dumping, PowerShell abuse)

### Skills Developed
- Windows security event log analysis
- Sysmon rule writing
- EDR alert triage
- Active Directory security concepts
- Windows-specific threat detection

### Career Mapping
Directly relevant to: SOC Analyst, Endpoint Security Engineer, Incident Responder roles

---

## Phase 4 — Automation and Infrastructure as Code

### Objective
Automate repeatable lab tasks using Ansible and Python, introducing Infrastructure-as-Code principles into the security environment.

### What Will Be Built
- Ansible playbooks for:
  - VM provisioning on Proxmox
  - Security tool installation (Wazuh agent, Suricata)
  - Firewall rule deployment
  - System hardening (CIS benchmark automation)
- Python scripts for:
  - Parsing and summarising Suricata/Wazuh alerts
  - Automated vulnerability scan reporting (Nessus API)
  - Lab health check and status reporting

### Skills Developed
- Infrastructure automation
- Python scripting for security operations
- Ansible playbook development
- API integration
- DevSecOps fundamentals

### Career Mapping
Directly relevant to: Security Engineer, DevSecOps Engineer, Cloud Security Engineer roles

---

## Phase 5 — Public Portfolio and GitHub Pages Site

### Objective
Convert this GitHub repository into a fully public, professionally presented portfolio with a companion website — making all lab work easily accessible to hiring managers and technical interviewers.

### What Will Be Built
- This repository fully completed with all phase documentation
- GitHub Pages portfolio website with:
  - Lab architecture overview
  - Project showcase with screenshots
  - Skills and certifications section
  - Links to write-ups, reports, and documentation
- Formal penetration test report (lab exercise)
- Formal incident response report (simulated scenario)
- Vulnerability assessment report (Nessus findings)

### Career Mapping
Public proof of work — directly relevant to every security role application

---

## Timeline

| Phase | Target | Status |
|:---|:---|:---|
| Foundation (current) | Completed | ✅ |
| Phase 1 — Wazuh SIEM | Q2 2025 | 🔄 In progress |
| Phase 2 — Kali Attack Simulation | Q3 2025 | 📅 Planned |
| Phase 3 — Windows Endpoint | Q3 2025 | 📅 Planned |
| Phase 4 — Automation | Q4 2025 | 📅 Planned |
| Phase 5 — Public Portfolio Site | Q4 2025 | 📅 Planned |

---

## Why This Roadmap Matters

Each phase is not just a technical exercise. Each one adds a documented, demonstrable capability that answers a specific question an interviewer might ask:

| Interviewer Question | Phase That Answers It |
|:---|:---|
| "Have you worked with a SIEM?" | Phase 1 — Wazuh |
| "Can you perform a penetration test?" | Phase 2 — Kali |
| "Do you understand endpoint security?" | Phase 3 — Windows/EDR |
| "Are you comfortable with automation?" | Phase 4 — Ansible/Python |
| "Can I see your work?" | Phase 5 — Public portfolio |

---

*Previous: [Troubleshooting Log](07-troubleshooting-log.md) | [← Back to README](../README.md)*
