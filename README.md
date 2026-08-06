# homelab-infrastructure

Personal homelab running 50+ endpoints across Windows, macOS, and Linux — used for hands-on testing of endpoint security configurations, MDM policies, identity infrastructure, and network segmentation.

This repo documents the topology, configuration references, and validation notes I use when testing enterprise security concepts in a controlled environment.

## Infrastructure Overview

```
┌─────────────────────────────────────────────────────┐
│              UniFi Dream Machine Pro (UDM Pro)        │
│         Router + Firewall + IDS/IPS + VLAN routing    │
└───┬───────┬───────┬───────┬───────┬───────┬──────────┘
    │       │       │       │       │       │
  VLAN 01 VLAN 10 VLAN 20 VLAN 30 VLAN 40 VLAN 50/60
   Base   HomeWired HomeLab Wireless  IoT   Sec+/Kids
    │       │       │
 native  R820/iDRAC Synology NAS
 trunk   Flex Mini   + Honeypot
         Raspberry Pi (10.10.20.8)
```

## Core Components

| Component | Platform | Purpose |
|-----------|----------|---------|
| Router / Firewall | UniFi Dream Machine Pro (UDM Pro) | VLAN routing, firewall rules, IDS/IPS |
| Switching | UniFi USW Flex Mini | Managed switching for HomeWired segment |
| Server | Dell PowerEdge R820 | Hypervisor host (iDRAC-managed) |
| Storage | Synology NAS | Network storage, HomeLab segment |
| Compute | Raspberry Pi | Lightweight services, SSH key-only access |
| MDM (Windows) | Microsoft Intune | Policy enforcement, compliance baselines |
| MDM (macOS) | Jamf Pro | macOS device management, configuration profiles |
| Identity | Okta / Active Directory | SSO, MFA, conditional access |
| Deception | Honeypot (10.10.20.8) | Active canary, IDS-monitored |
| Endpoint OS | Windows 10/11, macOS 14/15, Ubuntu | Policy targets |

## Network Topology

### VLANs (UniFi UDM Pro — 8-network structure)
| VLAN | Name | Purpose |
|------|------|---------|
| 01 | Base | Native/untagged trunk |
| 10 | HomeWired (tagged 1010) | Wired infra: R820, USW Flex Mini, Raspberry Pi |
| 20 | HomeLab | Synology NAS, honeypot, lab/testing segment |
| 30 | Wireless | General Wi-Fi clients |
| 40 | IoT | Isolated network for untrusted/IoT devices |
| 50 | Sec+ | Security devices (⚠️ config mismatch — labeled 50, actually tagged VLAN 47) |
| 60 | Kids Only | Kids' devices, isolated segment |

### Firewall / Segmentation Notes
- UDM Pro running IDS/IPS mode across all VLANs
- Honeypot deployed at 10.10.20.8 (HomeLab VLAN) as an active canary — IDS-monitored
- Raspberry Pi (10.10.10.10) restricted to public-key SSH only, no password auth
- UDM Pro Port 5 → USW Flex Mini / R820, native VLAN HomeWired

### Known Gaps / In-Progress Items
- Dell PowerEdge R820 iDRAC (10.10.10.13) currently blocked by NIC IP filter — requires physical console access (F2 → iDRAC Settings → disable IP filter) to resolve
- Unidentified device ("Dell 7k") present on UDM Pro Port 2 — pending identification
- VLAN 50 (Sec+) has a labeling/tagging mismatch between the UniFi UI (50) and the actual VLAN tag in use (47) — flagged for cleanup

This lab is actively maintained and evolving — the gaps above are tracked, not hidden, because real environments have them too.

## MDM Policy Testing

### Intune (Windows)
- Testing MDM CSP configurations via OMA-URI
- Validating WMI Bridge Provider behavior for CSP/GPO overlap
- Compliance policy enforcement and conditional access integration
- See [`/intune`](/intune) for policy exports and test notes

### Jamf Pro (macOS)
- Configuration profile testing (mobileconfig)
- Smart Group targeting and scoping
- Extension Attributes for custom compliance checks
- See [`/jamf`](/jamf) for profile templates

## Security Validation Workflow

When testing a new configuration:

1. **Snapshot** the target VM before applying changes
2. **Apply** policy via MDM or GPO
3. **Validate** expected behavior (registry keys, service state, log entries)
4. **Document** actual vs. expected behavior, edge cases, fallback behavior
5. **Revert** snapshot if needed, iterate

This mirrors the validation workflow I used at Tanium validating CSPs across enterprise environments.

## Directory Structure

```
homelab-infrastructure/
├── README.md
├── network/
│   ├── topology.md          # Full network diagram + VLAN table
│   ├── udm-pro-rules.md     # UDM Pro firewall rule documentation
│   └── vlan-config.md       # VLAN configuration reference
├── intune/
│   ├── README.md
│   ├── csp-test-notes.md    # CSP validation findings
│   ├── compliance-policies/ # Exported compliance policy JSON
│   └── config-profiles/     # Configuration profile exports
```

## Why This Exists

Enterprise security tools behave differently in the real world than in documentation. This lab exists to find those gaps — undocumented CSP interactions, GPO fallback edge cases, policy conflicts between MDM and Group Policy, certificate chain failures — before they become production incidents.

---

*Maintained by Kofi Asirifi · [LinkedIn](https://linkedin.com/in/kofiasirifi)*
