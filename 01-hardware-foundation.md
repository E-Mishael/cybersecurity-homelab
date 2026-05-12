# 01 — Hardware Foundation

## Overview

This homelab runs entirely on a single repurposed business laptop. The decision to use existing hardware rather than purpose-built server equipment was deliberate — it forces resource-conscious design decisions and proves that enterprise-grade security skills can be developed without expensive infrastructure.

---

## Physical Hardware

| Component | Specification | Relevance |
|:---|:---|:---|
| **Device** | HP EliteBook 840 G3 | Business-grade durability, passive cooling |
| **CPU** | Intel Core i5-6200U / i7-6600U | Supports VT-x and VT-d (hardware virtualization) |
| **RAM** | 16 GB DDR4 | Sufficient for 3–4 concurrent lightweight VMs |
| **Storage** | 256 GB SSD | Fast I/O for virtual machine disks |
| **Network** | 1× Gigabit Ethernet NIC | Single physical interface — drives creative network design |
| **BIOS** | UEFI with VT-x enabled | Required for Type-1 hypervisor operation |

---

## Physical Setup

The laptop connects to the home router via a single Ethernet cable. All internal virtual networking is handled through Proxmox Linux bridges — no additional physical switches or NICs required.

![Physical Ethernet Connection](../images/04_hardware_ethernet.jpg)
*HP EliteBook 840 G3 connected via Ethernet to the home router — the single physical link that carries all lab traffic.*

![Router Connection](../images/05_router_physical.jpg)
*Home router with the lab Ethernet cable visible — the physical boundary between the home network and the lab environment.*

---

## Why Single-NIC Matters

Most homelab tutorials assume multiple network interface cards. This build uses only one, which required designing a proper dual-bridge virtual network in Proxmox to maintain segmentation — a constraint that produced a stronger, more realistic architecture than a simple multi-NIC setup would have.

This is not a limitation. It is a design challenge that was solved.

---

## Hardware Constraints and Design Decisions

| Constraint | Design Decision | Outcome |
|:---|:---|:---|
| 1 physical NIC | Dual Linux bridge (vmbr0/vmbr1) architecture | Clean segmentation without extra hardware |
| 16 GB RAM | Lightweight VM selection (Xubuntu over GNOME) | 3–4 VMs running concurrently |
| 256 GB SSD | LVM thin provisioning for VM disks | Efficient storage allocation |
| No dedicated GPU | All VMs run headless or lightweight GUI | No resource waste |

---

*Next: [Proxmox Setup →](02-proxmox-setup.md)*
