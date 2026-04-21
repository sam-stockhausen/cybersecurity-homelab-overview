# Overview
Deployed Sysmon (System Monitor) on DC01 and Win10-Client using the SwiftOnSecurity 
configuration for rich Windows telemetry. DC01 was installed manually; Win10-Client 
was deployed via Group Policy startup script through SYSVOL — simulating an 
enterprise-scale rollout. Both machines now generate structured event logs ready for 
Wazuh SIEM agent ingestion.

---

## Architecture
DC01 (Windows Server 2022)
├── Sysmon64 service — manual install
├── SwiftOnSecurity config
└── GPO: Sysmon Deployment
│
└── SYSVOL startup script
│
└── Win10-Client (Windows 10 Pro)
└── Sysmon64 service — GPO deployed

---

## Implementation

**Sysmon Installation on DC01**
- Downloaded Sysmon from Sysinternals via PowerShell to bypass browser EULA prompt
- Downloaded SwiftOnSecurity sysmonconfig-export.xml from GitHub
- Added C:\Tools\Sysmon to Windows Defender exclusions to prevent quarantine
- Installed Sysmon with config: `Sysmon64.exe -accepteula -i sysmonconfig-export.xml`
- Verified Sysmon64 service running and Operational log populating in Event Viewer

**GPO Deployment to Win10-Client**
- Placed Sysmon64.exe, sysmonconfig-export.xml, and install-sysmon.ps1 in the GPO's
  SYSVOL startup scripts folder for automatic replication
- Created PowerShell startup script to copy files locally and run silent install
- Linked GPO to domain root (homelab.local) so all domain computers inherit policy
- Enabled PowerShell script execution via GPO Administrative Templates
- Verified policy application with gpresult /r /scope computer

---

## Deployment Workflow

Install Sysmon manually on DC01
Verify telemetry in Event Viewer — Sysmon Operational log
Stage Sysmon64.exe + config + install script in SYSVOL startup folder
Create GPO linked to domain root with PowerShell startup script
Enable script execution policy via GPO
Force policy update and reboot Win10-Client
Verify Sysmon64 service running on Win10-Client
Confirm Operational log populating on Win10-Client


---

## Key Events Generated (SwiftOnSecurity Config)

| Event ID | Description |
|----------|-------------|
| 1 | Process creation |
| 3 | Network connection |
| 7 | Image/DLL loaded |
| 10 | Process access |
| 11 | File created |
| 13 | Registry value set |
| 22 | DNS query |

---

## Key Concepts Demonstrated
- Host-based telemetry collection with Sysmon
- Enterprise GPO deployment via SYSVOL startup scripts
- SwiftOnSecurity config for production-grade event filtering
- Windows event log structure and Operational log access
- Prereq infrastructure for SIEM agent ingestion (Wazuh)
- Group Policy troubleshooting — gpresult, OU scoping, link verification

---

## Challenges
- Windows Defender quarantines Sysmon on download — resolved by adding 
  C:\Tools\Sysmon to exclusions before extracting
- Sysinternals download page requires EULA acceptance — bypassed using 
  Invoke-WebRequest directly in PowerShell
- GPO initially linked to Domain Controllers OU instead of domain root — 
  Win10-Client was not receiving policy until link was corrected
- SYSTEM context used by startup scripts cannot access SMB shares restricted 
  to Domain Computers without explicit NTFS + share permissions — resolved by 
  staging files directly in SYSVOL instead of a custom share
- GUID in $source path must include curly braces and match exact casing — 
  Windows strips braces when resolving paths causing Copy-Item to fail

---

## Tools & Technologies
- Sysmon v15.x (Sysinternals)
- SwiftOnSecurity sysmonconfig-export.xml
- Group Policy Management Console (gpmc.msc)
- PowerShell — Invoke-WebRequest, Get-Service, gpresult
- Windows Event Viewer — Sysmon Operational log
- Active Directory Users and Computers (dsa.msc)
- SYSVOL replication for GPO file distribution
