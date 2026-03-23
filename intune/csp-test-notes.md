# Intune CSP Validation Notes

Testing MDM Configuration Service Providers via OMA-URI in Intune, with a focus on 
WMI Bridge Provider behavior and GPO interaction.

## Test Environment
- Windows 11 22H2 / 23H2
- Intune MDM enrolled (AAD joined)
- Some machines also domain-joined (hybrid) to test CSP/GPO conflicts

## Key Findings

### WMI Bridge Provider Behavior
When MDM policy is applied via CSP and Group Policy exists for the same setting:
- MDM-enrolled AAD-joined: CSP wins, GPO ignored
- Hybrid-joined: MDM wins for supported CSPs, GPO fallback for unsupported
- Conflict detection: check `MDM Diagnostics` → `PolicyManager` node in registry

**Registry path for CSP validation:**
```
HKLM\SOFTWARE\Microsoft\PolicyManager\current\device\{Area}\{Policy}
```

### Defender CSP (./Vendor/MSFT/Defender)
| Setting | CSP Path | Expected | Actual | Notes |
|---------|----------|----------|--------|-------|
| TamperProtection | ./Device/Vendor/MSFT/Defender/Configuration/TamperProtection | 1 (enabled) | 1 | ✓ |
| DisableRealtimeMonitoring | ./Device/Vendor/MSFT/Policy/Config/Defender/AllowRealtimeMonitoring | 0 | 0 | ✓ |
| CloudBlockLevel | ./Device/Vendor/MSFT/Defender/Configuration/CloudBlockLevel | 2 | 2 | ✓ |

### BitLocker CSP
- Silent encryption requires AAD join + TPM 2.0 + Modern Standby
- Hybrid-joined machines: silent encryption fails if GPO BitLocker policy exists
- Fix: remove conflicting GPO or use MDM win via `./Device/Vendor/MSFT/BitLocker/RequireDeviceEncryption`

### Known Conflicts
1. **Firewall rules**: Intune firewall CSP + GPO firewall rules merge (not override) — can cause unexpected allow rules
2. **Windows Update**: `Update/BranchReadinessLevel` via CSP conflicts with WSUS GPO on hybrid-joined machines
3. **AppLocker vs WDAC**: AppLocker policies via GPO and WDAC via MDM coexist but WDAC takes precedence

## Validation Script

```powershell
# Check MDM enrollment and CSP policy state
function Get-MDMPolicyState {
    param([string]$PolicyArea, [string]$PolicyName)
    
    $path = "HKLM:\SOFTWARE\Microsoft\PolicyManager\current\device\$PolicyArea"
    if (Test-Path $path) {
        $value = Get-ItemProperty -Path $path -Name $PolicyName -ErrorAction SilentlyContinue
        return $value.$PolicyName
    }
    return "Not set"
}

# Example usage
Get-MDMPolicyState -PolicyArea "Defender" -PolicyName "AllowRealtimeMonitoring"
Get-MDMPolicyState -PolicyArea "BitLocker" -PolicyName "RequireDeviceEncryption"
```
