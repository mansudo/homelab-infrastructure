# Network Topology

## Physical / Virtual Layout

```
Internet
    │
[pfSense] — Suricata IDS, OpenVPN
    │
[Core Switch] — 802.1Q VLAN trunk
    ├── VLAN 10 (Management)  — ESXi hosts, vCenter, domain controllers
    ├── VLAN 20 (Endpoints)   — Windows, macOS, Linux test endpoints
    ├── VLAN 30 (DMZ)         — Test servers, exposed services
    └── VLAN 40 (IoT)         — Isolated untrusted devices
```

## Subnets

| VLAN | Subnet | Gateway | DHCP Range |
|------|--------|---------|------------|
| 10 | 10.0.10.0/24 | 10.0.10.1 | Static only |
| 20 | 10.0.20.0/24 | 10.0.20.1 | 10.0.20.100–200 |
| 30 | 10.0.30.0/24 | 10.0.30.1 | 10.0.30.100–150 |
| 40 | 10.0.40.0/24 | 10.0.40.1 | 10.0.40.100–150 |

## Static Assignments (Management VLAN)

| Host | IP | Role |
|------|----|------|
| esxi-01 | 10.0.10.10 | Primary ESXi host |
| esxi-02 | 10.0.10.11 | Secondary ESXi host |
| vcenter | 10.0.10.20 | vCenter Server |
| dc-01 | 10.0.10.30 | Primary Domain Controller |
| dc-02 | 10.0.10.31 | Replica DC |
| ca-01 | 10.0.10.40 | Issuing Certificate Authority |
| wazuh | 10.0.10.50 | SIEM / Log aggregation |
