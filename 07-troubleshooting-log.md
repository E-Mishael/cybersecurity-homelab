# 07 — Troubleshooting Log

## Overview

This document records every significant problem encountered during the build of this homelab, along with the root cause and the steps taken to resolve it. It is one of the most valuable sections of this portfolio.

Anyone can follow a tutorial that works perfectly. Real engineering skill is demonstrated by diagnosing failures, understanding root causes, and building correctly the second time. Every entry here represents a real obstacle that was independently researched and resolved.

---

## Issue Index

| # | Problem | Component | Severity |
|:---|:---|:---|:---|
| 1 | USB wouldn't boot Proxmox installer | BIOS / Hardware | High |
| 2 | Proxmox `apt update` failing after install | Repository config | Medium |
| 3 | pfSense installer stuck at loader screen | ISO compatibility | High |
| 4 | pfSense only detected one network interface | VM hardware config | High |
| 5 | pfSense web GUI unreachable after config | Interface assignment | High |
| 6 | Lab VM getting IP from home router instead of pfSense | Bridge assignment | Medium |
| 7 | Proxmox web GUI showing certificate warning | Self-signed TLS cert | Low |

---

## Issue 1 — USB Would Not Boot Proxmox Installer

### Symptom
After flashing the Proxmox VE ISO to a USB drive and setting USB as the first boot device, the laptop either booted directly to Windows or displayed a "No bootable device found" error.

### Root Cause
Two separate causes were identified:

1. **Fast Boot** was enabled in BIOS — this feature skips USB device enumeration during POST to reduce boot time, preventing bootable USB drives from being detected.
2. **Secure Boot** was enabled — Proxmox's bootloader is not signed with a Microsoft-trusted certificate, causing Secure Boot to reject it.

### Resolution
1. Entered BIOS (F10 on HP EliteBook during POST)
2. Disabled **Fast Boot** under Boot Options
3. Disabled **Secure Boot** under Security settings
4. Set boot order: USB first, SSD second
5. Saved and rebooted — USB was detected and Proxmox installer loaded successfully

### Lesson
Always verify BIOS settings before assuming a USB flash failed. Fast Boot is a common silent culprit on business laptops.

---

## Issue 2 — Proxmox `apt update` Errors After Installation

### Symptom
Running `apt update` immediately after Proxmox installation produced the following error:

```
Err:1 https://enterprise.proxmox.com/debian/pve bookworm InRelease
  401 Unauthorized [IP: x.x.x.x 443]
W: Failed to fetch https://enterprise.proxmox.com/debian/pve bookworm InRelease
   401 Unauthorized
```

### Root Cause
Proxmox VE ships with the **enterprise subscription repository** enabled by default. This repository requires a paid subscription key. Without it, all package operations fail with a 401 Unauthorized error.

### Resolution

```bash
# Step 1: Remove the enterprise repo
rm /etc/apt/sources.list.d/pve-enterprise.list

# Step 2: Add the no-subscription community repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" \
  > /etc/apt/sources.list.d/pve-no-subscription.list

# Step 3: Optionally disable the Ceph enterprise repo too
echo "# deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise" \
  > /etc/apt/sources.list.d/ceph.list

# Step 4: Update and upgrade
apt update && apt dist-upgrade -y
```

### Lesson
This is a known and expected post-install step for homelab use. The enterprise repository is enabled by default but non-functional without a subscription. This is not a bug — it is intentional commercial design.

---

## Issue 3 — pfSense Installer Stuck at Loader Screen

### Symptom
After booting the pfSense CE DVD ISO, the installer reached the FreeBSD boot loader screen and froze — no progress after 5+ minutes. The cursor stopped blinking.

### Root Cause
The standard pfSense DVD ISO uses a graphical boot process that has known compatibility issues with certain virtualized hardware configurations, particularly with OVMF (UEFI) and the q35 machine type in Proxmox.

### Resolution
Switched from the DVD ISO to the **pfSense memstick (nano) image**, which is a pre-installed image that bypasses the installer entirely:

1. Downloaded pfSense CE memstick serial image
2. Created a new VM in Proxmox configured for the nano image format
3. Wrote the image directly to the VM disk using Proxmox shell:

```bash
# Write pfSense nano image directly to VM disk
dd if=pfSense-CE-memstick-serial-2.7.2-RELEASE-amd64.img \
   of=/dev/pve/vm-100-disk-0 bs=1M status=progress
```

4. Started the VM — pfSense booted directly into the console without an installer

### Lesson
Always check whether a pre-built image exists before troubleshooting installer compatibility. The nano/memstick image is specifically designed for virtual environments.

---

## Issue 4 — pfSense Only Detected One Network Interface

### Symptom
During pfSense initial setup, the console showed only one available network interface (`vtnet0`), but two were needed — one for WAN and one for LAN.

### Root Cause
When the pfSense VM was initially created in Proxmox, only one network device was added. The VM hardware configuration was missing the second virtual NIC.

### Resolution
1. Shut down the pfSense VM
2. In Proxmox web GUI → VM 100 → Hardware → Add → Network Device
3. Added second network device: `vtnet1`, bridge `vmbr1`, model VirtIO
4. Started VM — pfSense now detected two interfaces: `vtnet0` and `vtnet1`
5. Assigned `vtnet0` → WAN, `vtnet1` → LAN

### Lesson
VM hardware must be fully configured before OS installation or initial setup. Always confirm the number of virtual NICs matches the intended network design before starting a firewall VM.

---

## Issue 5 — pfSense Web GUI Unreachable After Initial Config

### Symptom
After completing pfSense interface assignment and setting the LAN IP to `192.168.1.1`, the web GUI at `https://192.168.1.1` was completely unreachable from the Ubuntu VM on the same network.

### Root Cause
The interfaces were assigned in reverse — `vtnet0` was assigned to LAN and `vtnet1` was assigned to WAN. This meant:

- The "LAN" interface was on `vmbr0` (connected to the home router) — not the internal network
- The "WAN" interface was on `vmbr1` (internal bridge) — effectively blocking itself
- No VM on `vmbr1` could reach pfSense because pfSense's LAN was on the wrong bridge

### Resolution
1. Accessed pfSense via the physical console (Proxmox VM console)
2. Selected option 1 — Assign Interfaces
3. Reassigned correctly: `vtnet0` → WAN (vmbr0), `vtnet1` → LAN (vmbr1)
4. Reset LAN IP to `192.168.1.1/24`
5. Web GUI became immediately accessible from Ubuntu VM

### Lesson
Interface assignment is the most critical step in firewall setup. Always draw the network diagram first and verify which virtual NIC maps to which Proxmox bridge before assigning roles.

---

## Issue 6 — Ubuntu VM Getting IP from Home Router Instead of pfSense

### Symptom
After the Ubuntu VM was provisioned, it received an IP address in the `192.168.100.x` range instead of the expected `192.168.1.x` range from pfSense's DHCP server. This meant Ubuntu was on the home network — completely bypassing pfSense.

### Root Cause
During VM creation, Ubuntu's network adapter was attached to `vmbr0` (the external bridge connected to the home router) instead of `vmbr1` (the internal bridge behind pfSense).

### Resolution
1. Shut down Ubuntu VM
2. Proxmox web GUI → Ubuntu VM → Hardware → Network Device → Edit
3. Changed bridge from `vmbr0` to `vmbr1`
4. Started VM — Ubuntu obtained IP `192.168.1.x` from pfSense DHCP
5. Verified routing: `ip route` showed `192.168.1.1` as default gateway
6. Confirmed internet access routed through pfSense

### Lesson
Every VM intended to be in the lab network must be attached to the internal bridge (`vmbr1`) — not the external bridge. This is easy to overlook during VM creation and has significant security implications if missed.

---

## Issue 7 — Proxmox Web GUI Certificate Warning

### Symptom
Every time the Proxmox web GUI was accessed at `https://192.168.100.2:8006`, the browser displayed a "Your connection is not private" / certificate warning.

### Root Cause
Proxmox generates a self-signed TLS certificate during installation. Browsers do not trust self-signed certificates because they are not issued by a recognised Certificate Authority (CA).

### Resolution (Accepted Risk)
For a homelab environment accessible only on the internal network, this is an accepted and documented risk. The connection is still encrypted — the browser warning means the certificate is not CA-verified, not that the connection is unencrypted.

Options evaluated:

| Option | Description | Chosen? |
|:---|:---|:---|
| Accept warning permanently | Add browser exception | ✅ Yes — practical for internal use |
| Generate Let's Encrypt cert | Requires public domain name | ❌ No — not worth for internal-only |
| Deploy internal CA | Issue private CA cert to Proxmox | Planned future enhancement |

### Lesson
Self-signed certificates are acceptable on isolated internal management interfaces. Documenting the conscious decision to accept the risk is more professional than silently ignoring the warning.

---

*Previous: [WireGuard VPN](06-wireguard-vpn.md) | Next: [Roadmap →](08-roadmap.md)*
