# 04 — pfSense Firewall Configuration

## Overview

pfSense CE 2.7.2 is the firewall and network gateway for this lab. It runs as a dedicated virtual machine on Proxmox and handles all traffic between the internal lab network and the outside world. Every packet that leaves a lab VM passes through pfSense — no exceptions.

pfSense is a professional-grade, open-source firewall platform used in real enterprise environments. Configuring it from scratch in this lab provides direct transferable experience to commercial firewall platforms such as Fortinet, Palo Alto, and Cisco ASA.

---

## VM Specifications

| Parameter | Value |
|:---|:---|
| VM ID | 100 |
| BIOS | OVMF (UEFI) |
| CPU | 1 vCPU (host type) |
| RAM | 4 GB (4096 MB) |
| Disk | 20 GB (virtio, qcow2, iothread enabled) |
| Network | vtnet0 → vmbr0 (WAN), vtnet1 → vmbr1 (LAN) |
| Node | mishaelproxmox |

---

## Interface Assignment

After installation, pfSense presents a console interface assignment prompt. Interfaces were assigned as follows:

| pfSense Interface | Virtual NIC | Proxmox Bridge | Network | Role |
|:---|:---|:---|:---|:---|
| WAN | vtnet0 | vmbr0 | 192.168.100.0/24 | Receives DHCP from home router |
| LAN | vtnet1 | vmbr1 | 192.168.1.0/24 | Internal lab gateway |

> **Critical step:** Assigning the wrong interface to WAN/LAN at this stage causes the firewall to be unreachable. This mistake was made once and documented in the [troubleshooting log](07-troubleshooting-log.md).

---

## LAN Configuration

The pfSense LAN interface was configured with a static IP and DHCP server:

| Setting | Value |
|:---|:---|
| LAN IP Address | `192.168.1.1/24` |
| DHCP Server | Enabled |
| DHCP Range | `192.168.1.100` – `192.168.1.200` |
| DNS Forwarder | Enabled (forwards to upstream DNS) |

Lab VMs receive their IP addresses automatically from pfSense's DHCP server and use pfSense as their default gateway and DNS resolver.

---

## Firewall Rules

### Default Policy — Deny All

pfSense's default behaviour on the WAN interface is to deny all inbound traffic not matching an established session. This default-deny posture means:

- No unsolicited inbound connections reach lab VMs
- All permitted traffic must be explicitly defined
- Attackers scanning the WAN IP find no open ports

### LAN Rules

A single rule permits all outbound traffic from the LAN to any destination:

| # | Action | Source | Destination | Protocol | Purpose |
|:---|:---|:---|:---|:---|:---|
| 1 | Allow | LAN net | Any | Any | Permit lab VMs to reach internet |

This rule mirrors the standard "allow internal, deny external" baseline found in most enterprise perimeter firewall configurations.

### WAN Rules

| # | Action | Source | Destination | Protocol | Purpose |
|:---|:---|:---|:---|:---|:---|
| 1 | Allow | Any | WAN IP:51820 | UDP | WireGuard VPN inbound |
| 2 | Block | Any | Any | Any | Default deny (implicit) |

---

## NAT Configuration

Outbound NAT is configured in automatic mode. pfSense automatically translates source IPs from `192.168.1.0/24` to the WAN IP before forwarding traffic to the home router.

**Port Forward (WireGuard VPN):**

| External Port | Protocol | Internal Destination | Purpose |
|:---|:---|:---|:---|
| UDP 51820 | UDP | pfSense WAN IP | WireGuard VPN tunnel |

This is the only port forwarded from the internet into the lab. All other ports remain blocked.

---

## DNS Configuration

- DNS Resolver (Unbound) enabled on pfSense LAN
- Forwards DNS queries upstream to the home router / ISP DNS
- All lab VMs use `192.168.1.1` as their DNS server
- Provides centralized DNS logging — useful for detecting suspicious lookups

---

## Web GUI Access

The pfSense web interface is accessible only from within the lab network:

```
https://192.168.1.1
```

It is not accessible from the home network or internet. Default credentials were changed immediately after installation.

---

## Security Hardening Applied

- Admin password changed from default
- HTTPS enforced on web GUI
- SSH access disabled (console-only management when not using web GUI)
- No management interfaces exposed to WAN
- Bogon networks blocked on WAN interface
- Private networks blocked on WAN interface (RFC1918 anti-spoofing)

---

## Relationship to Enterprise Firewalls

The concepts configured in pfSense map directly to enterprise platforms:

| pfSense Concept | Enterprise Equivalent |
|:---|:---|
| Interface assignment (WAN/LAN) | Physical/logical interface binding |
| Default deny on WAN | Implicit deny at perimeter |
| LAN allow rule | Internal trust zone policy |
| Outbound NAT | PAT / source NAT on enterprise firewall |
| Port forward | DNAT / destination NAT |
| DNS Resolver | Internal recursive DNS |
| Firewall rule logging | SIEM log source |

---

*Previous: [Network Architecture](03-network-architecture.md) | Next: [Suricata IDS/IPS →](05-suricata-ids.md)*
