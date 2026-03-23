# homelab-infrastructure

Personal homelab running 50+ endpoints across Windows, macOS, and Linux — used for hands-on testing of endpoint security configurations, MDM policies, identity infrastructure, and network segmentation.

This repo documents the topology, configuration references, and validation notes I use when testing enterprise security concepts in a controlled environment.

## Infrastructure Overview

```
┌─────────────────────────────────────────────────────┐
│                    pfSense Firewall                  │
│            (VLAN routing + IDS/IPS)                 │
└──────────┬──────────────┬──────────────┬────────────┘
           │              │              │
      VLAN 10          VLAN 20       VLAN 30
    Management        Endpoints      DMZ/Lab
           │              │              │
    vCenter/ESXi    Win/Mac/Linux    Test Servers
    Domain Ctrl      (50+ nodes)    (CA, SIEM, etc.)
```

## Core Components

| Component | Platform | Purpose |
|-----------|----------|---------|
| Hypervisor | VMware vCenter / ESXi | VM management, snapshots, lab isolation |
| Firewall | pfSense | VLAN segmentation, firewall rules, IDS |
| Domain Controller | Windows Server | AD DS, Group Policy, DNS, DHCP |
| Certificate Authority | Windows Server CA | Internal PKI, cert-based auth |
| MDM (Windows) | Microsoft Intune | Policy enforcement, compliance baselines |
| MDM (macOS) | Jamf Pro | macOS device management, configuration profiles |
| Identity | Okta | SSO, MFA, conditional access |
| SIEM | Wazuh | Log aggregation, alert correlation |
| Endpoint OS | Windows 10/11, macOS 14/15, Ubuntu | Policy targets |

## Network Topology

### VLANs
| VLAN | Name | Purpose |
|------|------|---------|
| 10 | Management | Hypervisor management, domain controllers |
| 20 | Endpoints | Windows/macOS/Linux test machines |
| 30 | DMZ | Exposed services, external-facing tests |
| 40 | IoT | Isolated network for untrusted devices |
| 99 | Native | Untagged/trunk |

### Firewall Rules (key)
- Management VLAN: strict allow-list, no internet except update sources
- Endpoints VLAN: internet allowed, blocked from Management
- Inter-VLAN routing: explicit rules only, deny-all default
- IDS: Suricata on pfSense, ET Open ruleset

## Domain Infrastructure

```
Domain: lab.local
├── Domain Controllers (2 — primary + replica)
├── Group Policy Objects
│   ├── Baseline Security (CIS L1)
│   ├── Defender Configuration
│   ├── BitLocker Enforcement
│   └── Audit Policy
├── Certificate Authority
│   ├── Root CA (offline)
│   └── Issuing CA (online)
└── DNS / DHCP
```

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

## CIS Benchmark Coverage

Actively testing against CIS Benchmarks:
- CIS Microsoft Windows 11 Benchmark v3.0
- CIS Microsoft Intune for Windows 11 Benchmark v3.0
- CIS macOS Sonoma Benchmark v1.0

See [`/cis-validation`](/cis-validation) for test scripts and results.

## Directory Structure

```
homelab-infrastructure/
├── README.md
├── network/
│   ├── topology.md          # Full network diagram + VLAN table
│   ├── pfsense-rules.md     # Firewall rule documentation
│   └── vlan-config.md       # VLAN configuration reference
├── domain/
│   ├── gpo-baseline.md      # Group Policy baseline documentation
│   ├── ca-setup.md          # Certificate Authority setup notes
│   └── ad-structure.md      # OU structure and delegation
├── intune/
│   ├── README.md
│   ├── csp-test-notes.md    # CSP validation findings
│   ├── compliance-policies/ # Exported compliance policy JSON
│   └── config-profiles/     # Configuration profile exports
├── jamf/
│   ├── README.md
│   ├── config-profiles/     # mobileconfig templates
│   └── smart-groups.md      # Smart Group logic
├── cis-validation/
│   ├── README.md
│   ├── windows-11/          # CIS L1/L2 validation scripts
│   └── macos/               # macOS benchmark checks
└── siem/
    ├── README.md
    └── wazuh-rules.md       # Custom detection rules
```

## Why This Exists

Enterprise security tools behave differently in the real world than in documentation. This lab exists to find those gaps — undocumented CSP interactions, GPO fallback edge cases, policy conflicts between MDM and Group Policy, certificate chain failures — before they become production incidents.

---

*Maintained by Kofi Asirifi · [LinkedIn](https://linkedin.com/in/kofiasirifi)*
