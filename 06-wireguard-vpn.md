# 06 — WireGuard VPN

## Overview

WireGuard is a modern, high-performance VPN protocol configured on pfSense to provide secure remote access into the lab network. It allows a connected client (laptop, phone) to tunnel directly into the `192.168.1.0/24` lab network from any external location — coffee shop, work, or anywhere with internet.

WireGuard was chosen over OpenVPN for this remote access use case because of its simpler configuration, faster handshake, and significantly lower resource overhead — while providing equivalent or better cryptographic security.

---

## Why WireGuard

| Feature | WireGuard | OpenVPN |
|:---|:---|:---|
| Protocol | UDP only | TCP or UDP |
| Handshake speed | ~100ms | ~500ms+ |
| Configuration complexity | Minimal | Moderate |
| Cryptography | Modern (ChaCha20, Curve25519) | Configurable (older defaults available) |
| Performance | Very high | Moderate |
| Code size | ~4,000 lines | ~70,000+ lines |

Smaller codebase = smaller attack surface. WireGuard's cryptographic choices are modern and non-negotiable — it cannot be misconfigured to use weak ciphers.

---

## Architecture

```
External Client (laptop/phone)
         │
         │  WireGuard tunnel (UDP 51820, encrypted)
         │
         ▼
Home Router — port forward UDP 51820 → pfSense WAN
         │
         ▼
pfSense WAN (192.168.100.x)
         │
    [WireGuard tunnel terminates here]
         │
         ▼
pfSense WireGuard interface (192.168.2.1/24)
         │
         ▼
Lab Network (192.168.1.0/24)
         │
    [Client can now access all lab VMs]
```

---

## Configuration — pfSense Side

### Tunnel Settings

| Parameter | Value |
|:---|:---|
| Listen Port | UDP 51820 |
| Tunnel Network | `192.168.2.0/24` (VPN subnet) |
| Interface IP | `192.168.2.1/32` (pfSense VPN endpoint) |

### Key Generation

WireGuard uses asymmetric cryptography. A key pair is generated for each peer:

```bash
# Generate server private key
wg genkey | tee server_private.key | wg pubkey > server_public.key

# Generate client private key
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

Private keys remain on their respective devices and are never transmitted. Only public keys are exchanged.

### Peer Configuration (Client)

| Parameter | Value |
|:---|:---|
| Client Public Key | *(generated on client device)* |
| Allowed IPs | `192.168.2.2/32` (client VPN IP) |
| Keepalive | 25 seconds |

---

## Configuration — Client Side

The client configuration file (loaded into the WireGuard app on laptop or phone):

```ini
[Interface]
PrivateKey = <client_private_key>
Address = 192.168.2.2/24
DNS = 192.168.1.1

[Peer]
PublicKey = <server_public_key>
Endpoint = <home_router_public_ip>:51820
AllowedIPs = 192.168.1.0/24, 192.168.2.0/24
PersistentKeepalive = 25
```

`AllowedIPs` defines which traffic is routed through the tunnel. Only lab network traffic (`192.168.1.0/24`) and VPN subnet traffic (`192.168.2.0/24`) traverse the tunnel — home internet traffic routes normally. This is **split tunnelling**.

---

## Port Forwarding

For the tunnel to accept inbound connections, UDP port 51820 is forwarded from the home router to the pfSense WAN IP:

```
Home Router NAT Rule:
External: UDP 51820 → Internal: 192.168.100.x (pfSense WAN IP), UDP 51820
```

---

## Security Design

### What Is and Is Not Exposed

| Service | Exposed to Internet | Method |
|:---|:---|:---|
| WireGuard VPN | Yes — UDP 51820 only | Port forward |
| Proxmox Web GUI | No | Internal only |
| pfSense Web GUI | No | Internal only |
| Lab VMs | No | Internal only |
| SSH | No | Internal only |

**No management interface is directly accessible from the internet.** Remote management is only possible after authenticating through the WireGuard tunnel.

### Cryptographic Primitives

WireGuard uses the following by default — no configuration required:

| Function | Algorithm |
|:---|:---|
| Key exchange | Curve25519 (ECDH) |
| Symmetric encryption | ChaCha20 |
| Message authentication | Poly1305 |
| Hashing | BLAKE2s |
| Handshake | Noise protocol framework |

These are modern, peer-reviewed cryptographic primitives with no known practical weaknesses.

---

## Operational Use

Once connected, the VPN client:
- Receives IP `192.168.2.2` from the WireGuard tunnel
- Can access all lab VMs at `192.168.1.x`
- Can access pfSense GUI at `https://192.168.1.1`
- Can access Proxmox GUI at `https://192.168.100.2:8006`

This enables full remote lab management from anywhere without exposing any service directly to the internet.

---

## Relationship to Enterprise VPN

| Lab Configuration | Enterprise Equivalent |
|:---|:---|
| WireGuard on pfSense | GlobalProtect on Palo Alto, Cisco Secure Client, Fortinet SSL-VPN |
| Port forward UDP 51820 | Firewall policy permitting VPN protocol inbound |
| Split tunnelling | Enterprise split-tunnel VPN profiles |
| Pre-shared key pairs | PKI certificates in enterprise VPN |
| Peer authentication | Certificate-based or MFA authentication |

---

*Previous: [Suricata IDS/IPS](05-suricata-ids.md) | Next: [Troubleshooting Log →](07-troubleshooting-log.md)*
