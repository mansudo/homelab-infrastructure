# UniFi Network Configuration

## Controller Access
- **Domain:** hoa.local
- **Username:** stored in Keychain (`unifi-hoa`)
- **Access:** UniFi Network Application (self-hosted)

## UniFi Devices

| Device | Role | Notes |
|--------|------|-------|
| UDM/USG | Gateway / Router | VLAN routing, IDS/IPS, VPN |
| Core Switch | L2/L3 switching | 802.1Q VLAN trunking |
| Access Points | WiFi | SSID per VLAN, WPA3 |

## VLAN Configuration (UniFi)

Networks are defined in UniFi → Settings → Networks:

| Network Name | VLAN ID | Subnet | Purpose |
|-------------|---------|--------|---------|
| Management | 10 | 10.0.10.0/24 | Hypervisors, controllers, DCs |
| Endpoints | 20 | 10.0.20.0/24 | Test Windows/macOS/Linux machines |
| DMZ | 30 | 10.0.30.0/24 | Exposed services |
| IoT | 40 | 10.0.40.0/24 | Isolated untrusted devices |

## Firewall Policies (UniFi)

Key rules in UniFi → Firewall & Security:

| Rule | Action | Purpose |
|------|--------|---------|
| Endpoints → Management | Block | Prevent endpoint lateral movement to infra |
| IoT → All | Block | Full IoT isolation except internet |
| Management → Internet | Allow (allowlist) | Update sources only |
| DMZ → LAN | Block | DMZ cannot initiate connections inward |

## WiFi SSIDs

| SSID | VLAN | Security | Purpose |
|------|------|----------|---------|
| lab-endpoints | 20 | WPA3 | Test device wireless |
| lab-mgmt | 10 | WPA3-Enterprise | Management access |
| iot-isolated | 40 | WPA2 | IoT/untrusted devices |

## IDS/IPS

- Suricata enabled on WAN interface
- Ruleset: ET Open (updated daily)
- Alert logging: forwarded to Wazuh SIEM

## Backup & Config Export

UniFi configuration is backed up via:
1. Auto-backup: UniFi → System → Backups (weekly, retain 5)
2. Manual export before major changes
3. Config stored locally — do not commit to public repo
