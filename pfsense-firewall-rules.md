# pfSense Firewall Rules — Reference

This document provides a reference overview of the firewall rules configured in pfSense for this lab. Actual rule syntax and screenshots will be added as the lab is further documented.

---

## WAN Interface Rules

| Priority | Action | Protocol | Source | Destination | Port | Description |
|:---|:---|:---|:---|:---|:---|:---|
| 1 | Allow | UDP | Any | WAN Address | 51820 | WireGuard VPN inbound |
| 2 | Block | Any | Any | Any | Any | Default deny (implicit) |

**Notes:**
- All inbound traffic not matching rule 1 is silently dropped
- No ICMP (ping) is permitted inbound to WAN
- No SSH, HTTP, or HTTPS inbound on WAN

---

## LAN Interface Rules

| Priority | Action | Protocol | Source | Destination | Port | Description |
|:---|:---|:---|:---|:---|:---|:---|
| 1 | Allow | Any | LAN net | Any | Any | Permit all outbound from lab VMs |

**Notes:**
- This rule allows lab VMs full internet access
- Traffic still passes through Suricata IDS/IPS for inspection
- Future: restrict by destination and protocol as lab matures

---

## NAT Rules

### Outbound NAT
Mode: Automatic

Proxmox generates automatic outbound NAT rules translating all `192.168.1.0/24` traffic to the pfSense WAN IP before forwarding upstream.

### Port Forwards (Inbound NAT)

| External Interface | Protocol | External Port | Internal Destination | Internal Port | Description |
|:---|:---|:---|:---|:---|:---|
| WAN | UDP | 51820 | pfSense WAN IP | 51820 | WireGuard VPN |

---

## Future Rule Additions (Planned)

As the lab matures, the following rules will be added and documented:

- **Restrict LAN → WAN** by protocol (allow only HTTP/HTTPS/DNS, block all else)
- **Isolate Kali VM** to a separate VLAN with tightly controlled egress
- **Allow Wazuh agent traffic** from all VMs to Wazuh Manager on specific ports
- **Block inter-VM traffic** by default (Zero Trust between lab segments)
- **IPS block list rules** auto-populated by Suricata blocked IPs

---

*See also: [pfSense Firewall Documentation](04-pfsense-firewall.md)*
