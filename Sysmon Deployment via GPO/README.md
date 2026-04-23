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

<img width="327" height="95" alt="getservice" src="https://github.com/user-attachments/assets/9dd0de21-8a59-4c49-8fbe-0e733c00c0f3" />
<img width="276" height="88" alt="getservice2" src="https://github.com/user-attachments/assets/bf343b51-4497-4a74-a91d-085f08fea45d" />
*Get-Service Sysmon64 output showing Running status on DC01*


**GPO Deployment to Win10-Client**
- Placed Sysmon64.exe, sysmonconfig-export.xml, and install-sysmon.ps1 in the GPO's
  SYSVOL startup scripts folder for automatic replication
- Created PowerShell startup script to copy files locally and run silent install
- Linked GPO to domain root (homelab.local) so all domain computers inherit policy
- Enabled PowerShell script execution via GPO Administrative Templates
- Verified policy application with gpresult /r /scope computer

<img width="432" height="596" alt="scope" src="https://github.com/user-attachments/assets/75ec3492-9f7f-47f9-b734-3f3bbadbc8c0" />
*gpresult /r /scope computer on Win10-Client showing Sysmon Deployment under Applied Computer GPOs*

---

## Deployment Workflow

Install Sysmon manually on DC01
Verify telemetry in Event Viewer — Sysmon Operational log

<img width="616" height="582" alt="eventviewer2" src="https://github.com/user-attachments/assets/a4380d7d-7fb1-44cb-8c95-21ff36487dd2" />
*Event Viewer on DC01 — Sysmon Operational log open with Event ID 1 entries visible*

Stage Sysmon64.exe + config + install script in SYSVOL startup folder
Create GPO linked to domain root with PowerShell startup script
Enable script execution policy via GPO
Force policy update and reboot Win10-Client
Verify Sysmon64 service running on Win10-Client
Confirm Operational log populating on Win10-Client

<img width="500" height="541" alt="eventviewer3" src="https://github.com/user-attachments/assets/18d9b914-a3f5-4be7-a2c1-f8d1e54e5a4f" />
*Event Viewer on Win10-Client — same view, proves GPO deployment produced identical telemetry*

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

<img width="685" height="362" alt="gpmc" src="https://github.com/user-attachments/assets/ddaacc20-ce39-4087-8bed-a013a3e72fdb" />
*gpmc.msc Scope tab — shows GPO linked to homelab.local root with Authenticated Users in Security Filtering*

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

<img width="544" height="107" alt="sysmonfolder" src="https://github.com/user-attachments/assets/d3a3b380-1810-48e8-b7a9-611af7d8ea76" />
*SYSVOL startup folder contents showing Sysmon64.exe, sysmonconfig-export.xml, and install-sysmon.ps1*
  
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
