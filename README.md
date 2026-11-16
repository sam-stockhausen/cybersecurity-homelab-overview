# Cybersecurity Homelab

A segmented, virtualized environment I use to practice infrastructure, network security and detection work, and to document what worked, what broke, and what I'd change.

## Timeline

- **v1 (May 2024):** First build alongside my Penn State internship. Linux fundamentals plus GoPhish phishing, brute-force and Wireshark MITM exercises.
- **v2 (Jan 2026 – present):** Rebuilt on Proxmox with VLAN segmentation, internal PKI, monitoring, host telemetry and a honeypot alerting pipeline.

**Status:** Active. Estimated hours: 150

---

## Architecture

Network diagram

(<img width="791" height="683" alt="network_diagram" src="https://github.com/user-attachments/assets/96c65cf9-1dc1-4f2d-90bd-a09c57382d5b" />)

### Hardware
- **Host:** HP EliteDesk 800 G6 SFF (Intel i5-10500, 16 GB RAM, 512 GB NVMe)
- **Switch:** TP-Link 5-port unmanaged
- **Hypervisor:** Proxmox VE 8.x

### Virtual machines

| Name | OS | Role | VLAN |
|---|---|---|---|
| pfSense | FreeBSD | Firewall, routing, WireGuard VPN | All |
| Ubuntu-Mgt | Ubuntu 24.04 | Jumpbox, Nginx Proxy Manager, OpenCanary, Loki/Promtail, scripts | 20 (Servers) |
| DC01 | Windows Server 2022 | Domain controller, DNS, Enterprise Root CA | 20 (Servers) |
| Monitoring | Ubuntu 24.04 | Prometheus, Grafana | 20 (Servers) |
| Pi-hole | Debian 12 | DNS and filtering | 20 (Servers) |
| Win10-Client | Windows 10 | Test workstation | 30 (Clients) |

VLAN 10 (Management) and VLAN 99 (DMZ) are configured but reserved for future hosts.

---

## Projects

### Security and detection
| Project | What it shows | Write-up |
|---|---|---|
| **OpenCanary honeypot pipeline** | Fake SSH/HTTP/RDP/SMB listeners → Promtail → Loki → Grafana alert that fires on internal probes | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/OpenCanary%20Honeypot/README.md) |
| **Sysmon via Group Policy** | SwiftOnSecurity config deployed to a domain client through a SYSVOL startup script; verified with gpresult and Event Viewer | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/Sysmon%20Deployment%20via%20GPO/README.md)|
| **Jumpbox (bastion) architecture** | Key-only SSH, firewall rules that block direct SSH to servers, ProxyJump | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/Jumpbox/README.md) |

### Infrastructure
| Project | What it shows | Write-up |
|---|---|---|
| **VLAN segmentation** | Proxmox VLAN-aware bridge, pfSense VLAN interfaces, default-deny inter-VLAN rules | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/VLAN%20Network%20Segmentation/README.md) |
| **Internal PKI (AD CS)** | Enterprise Root CA, SAN certificates, Windows and Linux trust distribution | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/Active%20Directory%20Certificate%20Services%20(PKI)/README.md) |
| **Nginx Proxy Manager** | Reverse proxy with a wildcard certificate from the internal CA, DNS routing via Pi-hole | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/Nginx%20Proxy%20Manager/README.md) |
| **WireGuard VPN** | Split-tunnel remote access to internal VLANs | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/WireGuard%20VPN/README.md) |
| **Pi-hole DNS** | Network-wide DNS filtering and local internal records | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/Pi-hole%20Network-Wide%20DNS%20%26%20Ad%20Blocking/README.md) |
| **Monitoring stack** | Prometheus, exporters and Grafana dashboards for every VM | [Write-up](https://github.com/sam-stockhausen/cybersecurity-homelab-overview/blob/main/Monitoring%20%26%20Observability%20Stack/README.md) |

### Automation
- **eyespy.py:** Python and nmap scanner that baselines devices across subnets and flags new ones 
- **update-vms.sh:** Bash script that patches Linux VMs over SSH and logs the results 

### v1 exercises (write-ups in progress)
GoPhish phishing campaign, brute-force password attack, Wireshark MITM and plaintext traffic analysis.

---

## Known limitations

- Single host with 16 GB RAM, which limits how many VMs run at once.
- The honeypot's portscan module is disabled because it needs raw socket access that Docker doesn't give it.
- The Management and DMZ VLANs have no hosts yet.
- Some certificate and key handling (for example, moving a PFX over a temporary HTTP server) is acceptable in a lab and would not be in production.

## Next

- Splunk Free with forwarders on the Sysmon hosts
- Atomic Red Team tests with detections mapped to MITRE ATT&CK
- Wazuh, once RAM allows

---

## Skills practiced

Proxmox, pfSense, VLAN design, Windows Server AD/DNS/Group Policy, AD CS/PKI, WireGuard, SSH hardening, Docker, Prometheus, Grafana, Loki, Sysmon, OpenCanary, Python, Bash

## Certifications

Security+ • CySA+ • CSAP • TryHackMe SOC Level 1 • Google Cybersecurity Professional • AZ-900 • CCSK

## Connect

[LinkedIn](https://www.linkedin.com/in/sam-stockhausen/)
