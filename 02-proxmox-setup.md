# 02 — Proxmox VE Setup

## Overview

Proxmox Virtual Environment (VE) 9.1.1 serves as the Type-1 hypervisor for this lab. It was installed directly on bare metal, replacing the laptop's original operating system. All virtual machines run on top of Proxmox using KVM (Kernel-based Virtual Machine) for full hardware virtualization.

---

## Why Proxmox

Proxmox VE is an open-source enterprise hypervisor used in production environments worldwide. Choosing it over consumer alternatives (VirtualBox, VMware Workstation) provides hands-on exposure to the same tooling used by IT and security teams in real organizations.

| Feature | Proxmox VE | Consumer Hypervisors |
|:---|:---|:---|
| Type | Type-1 (bare metal) | Type-2 (runs on OS) |
| Management | Web GUI + CLI | Desktop GUI |
| Networking | Linux bridges, VLANs | Basic virtual adapters |
| Storage | LVM, ZFS, NFS | Simple virtual disks |
| Industry use | Production environments | Personal/dev use |

---

## Installation Process

### 1. Pre-Installation Checks
Before installation, the following BIOS settings were verified and configured:
- **VT-x (Intel Virtualization Technology)** — enabled
- **Fast Boot** — disabled (caused USB boot failure, documented in troubleshooting log)
- **Boot order** — set to USB first, then SSD
- **Secure Boot** — disabled for compatibility

### 2. Installation Media
- Downloaded Proxmox VE 9.1.1 ISO from the official Proxmox website
- Flashed to USB using Rufus (Windows) / `dd` (Linux)
- Booted from USB into the Proxmox graphical installer

### 3. Network Configuration During Install

During installation, the following network parameters were assigned:

| Parameter | Value |
|:---|:---|
| Management Interface | nic0 — `80:ce:62:a7:a7:25` (e1000e) |
| Hostname (FQDN) | `mishael-proxmox-server` |
| IP Address | `192.168.100.2/24` |
| Gateway | `192.168.100.1` |
| DNS Server | `192.168.100.1` |

![Proxmox Installer Network Config](../images/03_proxmox_installer_network.jpg)
*Proxmox installer showing the management interface assignment and static IP configuration.*

### 4. Post-Installation Repository Fix

After installation, Proxmox attempted to contact the enterprise subscription repository, causing `apt update` errors. This was resolved by:

```bash
# Remove enterprise repository
rm /etc/apt/sources.list.d/pve-enterprise.list

# Add no-subscription community repository
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" \
  > /etc/apt/sources.list.d/pve-no-subscription.list

# Update and upgrade
apt update && apt dist-upgrade -y
```

> This is a common and expected step for homelab use. Documented here because it trips up most first-time Proxmox installers.

---

## Storage Configuration

Proxmox manages two storage pools:

| Pool | Type | Purpose |
|:---|:---|:---|
| `local` | Directory | ISO images, CT templates, backups |
| `local-lvm` | LVM-thin | Virtual machine disks (qcow2 format) |

LVM-thin provisioning allows VM disks to only consume actual space used, rather than pre-allocating the full declared size.

---

## Proxmox Dashboard

![Proxmox Dashboard](../images/07_proxmox_dashboard_2.jpg)
*Proxmox VE 9.1.1 web interface accessed at `https://192.168.100.2:8006` — showing the node `mishaelproxmox` and datacenter structure.*

![Proxmox Dashboard Detail](../images/06_proxmox_dashboard_1.jpg)
*Datacenter view showing the node, local storage, and local-lvm storage pools with current utilization.*

The web interface is accessible only from the internal home network at `https://192.168.100.2:8006`. It is not exposed to the internet.

---

## Virtual Machine Inventory

| VM ID | Name | vCPU | RAM | Disk | Role |
|:---|:---|:---|:---|:---|:---|
| 100 | pfSense | 1 | 4 GB | 20 GB | Firewall / Gateway |
| 101 | Ubuntu Server | 2 | 4 GB | 20 GB | Linux test server |
| *(planned)* | Kali Linux | 2 | 4 GB | 40 GB | Attack simulation |
| *(planned)* | Windows 10/11 | 2 | 4 GB | 60 GB | Endpoint testing |
| *(planned)* | Wazuh | 2 | 4 GB | 50 GB | SIEM |

---

## VM Creation — pfSense Example

The following screenshots show the actual VM configuration used for the pfSense firewall VM, confirming the real build parameters:

![VM Creation Config 1](../images/01_proxmox_vm_creation_1.jpg)
*Proxmox VM creation wizard — Confirm tab showing BIOS (OVMF/UEFI), CPU type, memory (4096 MB), disk, network bridge, and node assignment for the OPNsense/pfSense VM.*

![VM Creation Config 2](../images/02_proxmox_vm_creation_2.jpg)
*Full VM config including virtio disk (20 GB, qcow2), VM ID 100, and storage assignment on `mishaelproxmox` node.*

---

## Security Hardening Applied to Proxmox

- Default `root` password changed immediately after installation
- Web interface access restricted to local network only
- No Proxmox management ports forwarded to the internet
- Package repositories updated and system fully patched before VM deployment

---

*Next: [Network Architecture →](03-network-architecture.md)*
