# 05 — Suricata IDS/IPS

## Overview

Suricata is an open-source, high-performance Intrusion Detection and Prevention System (IDS/IPS) deployed directly within pfSense. It inspects all network traffic passing through the firewall in real time, comparing packets against a rule set to identify malicious or suspicious activity.

In IDS mode, Suricata generates alerts but does not block traffic. In IPS mode (inline), it actively drops packets matching threat signatures. This lab runs Suricata in IPS mode — mirroring enterprise deployments.

---

## Why Suricata Over Snort

Both Snort and Suricata are available as pfSense packages. Suricata was selected for this lab because:

- Multi-threaded architecture — better performance on modern CPUs
- Native support for EVE JSON logging — easier integration with SIEM platforms (Wazuh, Elastic)
- Actively maintained with frequent rule updates
- Industry standard in enterprise and SOC environments

---

## Deployment

Suricata is installed as a package directly within pfSense:

```
pfSense Web GUI → System → Package Manager → Available Packages → Suricata → Install
```

Once installed, it is configured per interface. In this lab, Suricata runs on the **WAN interface** to inspect all inbound and outbound traffic at the network perimeter.

---

## Rule Sets

Suricata uses rule sets — libraries of threat signatures — to identify malicious traffic. The following rule sets are enabled:

| Rule Set | Source | Purpose |
|:---|:---|:---|
| Emerging Threats Open | Proofpoint (free) | Broad coverage — malware, exploits, C2, scanning |
| Snort Community Rules | Snort / Cisco (free) | General network threat signatures |

Rules are updated on a scheduled basis to maintain current threat coverage.

---

## Interface Configuration

| Setting | Value |
|:---|:---|
| Interface | WAN (vtnet0) |
| Mode | IPS (inline — Legacy Mode) |
| Blocking Mode | Enabled |
| Alert Logging | Enabled (EVE JSON) |
| Block Offenders | Enabled — auto-blocks source IPs generating alerts |
| Block Duration | Configurable per rule |

---

## Alert Tuning — Reducing False Positives

A raw Suricata deployment with default rules generates a significant number of false positives — alerts on legitimate traffic. Tuning was performed to reduce noise while preserving detection of real threats:

### Tuning Methods Applied

**1. Pass Lists**
IP addresses known to be legitimate (home router, Proxmox host, lab VMs) are added to a pass list. Suricata skips these IPs during inspection, eliminating false alerts from internal management traffic.

**2. Suppression Rules**
Specific rule SIDs that consistently fire on known legitimate traffic are suppressed on a per-IP or per-network basis:

```
suppress gen_id 1, sig_id <SID>, track by_src, ip <trusted_ip>
```

**3. Rule Category Disabling**
Entire categories of rules that are not relevant to this environment (e.g., rules for services not running in the lab) are disabled to reduce unnecessary inspection overhead.

---

## What Suricata Detects in This Lab

| Threat Category | Example Signatures |
|:---|:---|
| Port scanning | Nmap SYN scan patterns, OS fingerprinting |
| Exploit attempts | Known CVE signatures, shellcode patterns |
| Malware C2 | Known command-and-control IP/domain signatures |
| DNS anomalies | DNS tunnelling, suspicious query patterns |
| Protocol abuse | Invalid protocol usage, evasion techniques |

---

## Log Output — EVE JSON

Suricata generates structured JSON logs in EVE (Extensible Event) format. This is the standard format consumed by SIEM platforms. A sample alert entry looks like:

```json
{
  "timestamp": "2025-03-15T14:23:11.456789+0000",
  "event_type": "alert",
  "src_ip": "203.0.113.45",
  "src_port": 54321,
  "dest_ip": "192.168.100.117",
  "dest_port": 22,
  "proto": "TCP",
  "alert": {
    "action": "blocked",
    "gid": 1,
    "signature_id": 2001219,
    "rev": 20,
    "signature": "ET SCAN Potential SSH Scan",
    "category": "Attempted Information Leak",
    "severity": 2
  }
}
```

When Wazuh SIEM is deployed (see [Roadmap](08-roadmap.md)), these logs will be ingested for centralized alerting, dashboards, and incident response workflows.

---

## Planned Enhancements

- Forward EVE JSON logs to Wazuh SIEM for centralized correlation
- Build custom detection rules for lab-specific attack simulations (Kali VM)
- Create weekly alert review process to simulate SOC analyst workflow
- Document specific attack → detection → response scenarios end to end

---

## Relationship to Enterprise Security Operations

| Lab Activity | Enterprise Equivalent |
|:---|:---|
| Deploying Suricata on pfSense | Deploying IDS/IPS on perimeter firewall |
| Tuning rules / suppressing false positives | SOC analyst — alert tuning and rule management |
| Reading EVE JSON alerts | SIEM alert triage |
| Blocking offending IPs | Automated threat response |
| Updating rule sets | Threat intelligence feed management |

---

*Previous: [pfSense Firewall](04-pfsense-firewall.md) | Next: [WireGuard VPN →](06-wireguard-vpn.md)*
