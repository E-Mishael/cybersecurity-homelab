# 03 — Network Architecture

## Overview

The lab network is built around a core security principle: **all traffic must pass through a firewall before reaching the internet.** This is achieved through a dual Linux bridge design in Proxmox, with pfSense acting as the sole gateway between the internal lab network and the outside world.

This architecture directly mirrors how enterprise environments isolate production, development, and guest networks from one another.

---

## Network Diagram

![Network Diagram](../images/08_network_diagram.png)

---

## The Two-Zone Design

The lab is split into two distinct network zones:

### Zone A — Home Network (External)
This is the existing home network provided by the home router. The Proxmox host and pfSense WAN interface sit in this zone.

| Device | IP Address | Role |
|:---|:---|:---|
| Home Router | `192.168.100.1` | Internet gateway, DHCP for home devices |
| Proxmox Host | `192.168.100.2` | Hypervisor management interface |
| pfSense WAN (vtnet0) | `192.168.100.x` (DHCP) | pfSense external-facing interface |

### Zone B — Lab Network (Internal / Isolated)
This is a completely isolated internal network that exists only inside Proxmox. Nothing in this zone can reach the internet without passing through pfSense first.

| Device | IP Address | Role |
|:---|:---|:---|
| pfSense LAN (vtnet1) | `192.168.1.1/24` | Gateway and DHCP server for all lab VMs |
| Ubuntu Server VM | `192.168.1.x` (DHCP) | Linux test server |
| Kali Linux *(planned)* | `192.168.1.x` | Attack simulation |
| Windows VM *(planned)* | `192.168.1.x` | Endpoint testing |
| Wazuh SIEM *(planned)* | `192.168.1.x` | Log collection and alerting |

---

## Proxmox Bridge Design

Proxmox uses **Linux bridges** as virtual switches. Each bridge connects VMs to either physical networks or isolated internal networks.

| Bridge | Physical NIC Attached | Purpose |
|:---|:---|:---|
| `vmbr0` | Yes — Gigabit Ethernet | Proxmox management + pfSense WAN traffic |
| `vmbr1` | No — internal only | pfSense LAN + all lab VMs |

### Why Two Bridges?

- **vmbr0** connects to the physical network card, giving Proxmox and pfSense's WAN interface access to the home router and internet.
- **vmbr1** is a completely internal virtual switch with no physical connection. Any VM attached to it has no path to the internet except through pfSense.

This means **network segmentation is enforced at the hypervisor level** — not just by firewall rules. A misconfigured firewall rule would still require traffic to pass through pfSense's routing stack. A VM cannot reach the internet by accident.

---

## Traffic Flow — Step by Step

### Outbound (VM → Internet)

```
VM on vmbr1 (e.g., 192.168.1.101)
        │
        ▼
pfSense LAN interface — vtnet1 (192.168.1.1)
        │
   [Firewall rules evaluated]
   [NAT applied — source IP translated]
        │
        ▼
pfSense WAN interface — vtnet0 (192.168.100.x)
        │
        ▼
Home Router (192.168.100.1)
        │
        ▼
      Internet
```

### What NAT Does Here

Network Address Translation (NAT) converts the private lab IP (`192.168.1.x`) into the pfSense WAN IP before traffic leaves. From the internet's perspective, all traffic originates from the router — internal lab IPs are completely hidden.

This serves two purposes:
1. **Security** — internal network topology is not exposed externally
2. **Functionality** — private IP ranges are not routable on the public internet

### Inbound (Internet → Lab)

No inbound connections are permitted by default. pfSense's default-deny policy blocks all unsolicited inbound traffic. The only exception is the WireGuard VPN tunnel (UDP 51820), which is explicitly port-forwarded and secured with cryptographic key pairs.

---

## Security Model

### Defense in Depth

| Layer | Control | Purpose |
|:---|:---|:---|
| Physical | Single Ethernet cable | Limits physical attack surface |
| Hypervisor | vmbr1 internal bridge | Enforces segmentation at VM level |
| Firewall | pfSense default-deny | Blocks all traffic not explicitly permitted |
| IDS/IPS | Suricata on pfSense | Inspects allowed traffic for threats |
| VPN | WireGuard (UDP 51820) | Only external access point — encrypted |

### Network Segmentation

The lab network (`192.168.1.0/24`) is completely isolated from the home network (`192.168.100.0/24`). Devices on the home network cannot initiate connections into the lab network, and lab VMs cannot reach home devices — all by design.

This mirrors enterprise network segmentation practices such as isolating development environments from production, or guest Wi-Fi from internal corporate resources.

---

## Mapping to Enterprise Concepts

| Lab Component | Enterprise Equivalent |
|:---|:---|
| `vmbr0` | Upstream provider / WAN link |
| `vmbr1` | Internal LAN / VLAN |
| pfSense VM | Perimeter firewall (Palo Alto, Fortinet, Cisco ASA) |
| NAT on pfSense | NAT Gateway (AWS) / PAT on enterprise firewall |
| DHCP on pfSense LAN | Internal DHCP server |
| WireGuard VPN | Site-to-site or remote access VPN |

---

*Previous: [Proxmox Setup](02-proxmox-setup.md) | Next: [pfSense Firewall →](04-pfsense-firewall.md)*
