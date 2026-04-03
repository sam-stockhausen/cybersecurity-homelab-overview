# cybersecurity-homelab-overview

A production-grade virtualized environment demonstrating enterprise IT and security practices

Project Overview

Built a complete enterprise homelab from scratch to develop hands-on skills in virtualization, network security, infrastructure automation, and security monitoring. This environment mirrors real-world corporate networks with proper segmentation, certificate management, and monitoring.

Timeline: January 2026 - Present  
Estimated Hours: 150+  
Status: Active Development



Architecture

Hardware
- Host: HP EliteDesk 800 G6 SFF
  - CPU: Intel i5-10500 (6 cores, 12 threads)
  - RAM: 16GB DDR4
  - Storage: 512GB NVMe SSD
- Network: TP-Link 5-port unmanaged switch

  

Virtual Machines

| Name         | OS                  | Role                         | VLAN |
|--------------|---------------------|------------------------------|------|
| pfSense      | FreeBSD             | Firewall, Router, VPN        | -    |
| Ubuntu-Mgt   | Ubuntu 24.04        | Management, Scripts, Jumpbox | 20   |
| DC01         | Windows Server 2022 | Domain Controller, DNS, CA   | 20   |
| Win10-Client | Windows 10          | Test Workstation             | 30   |
| Monitoring   | Ubuntu 24.04        | Prometheus, Grafana          | 20   |
| Pi-hole      | Debian 12           | DNS, Ad-blocking             | 20   |

---

Implemented Projects

1. Network Segmentation & VLAN Design
Technologies: pfSense, VLANs, inter-VLAN routing  
Skills: Network architecture, firewall configuration, security zones

- Designed 4-tier VLAN architecture (Management, Servers, Clients, DMZ)
- Configured VLAN-aware bridge in Proxmox
- Implemented inter-VLAN routing with firewall rules
- Separated production services from client devices

**Key Achievement:** Zero trust network architecture - each VLAN has specific access controls



2. Active Directory Certificate Services (PKI)
Technologies: AD CS, OpenSSL, SSL/TLS  
Skills: Public Key Infrastructure, certificate lifecycle management

- Deployed Enterprise Root CA on Windows Server 2022
- Generated SSL certificates with Subject Alternative Names (SAN)
- Implemented certificate distribution across Linux and Windows
- Configured Grafana for HTTPS with valid certificates

**Key Achievement:** Green padlock on all internal services - no browser warnings

**Challenges Overcome:**
- Windows certificate exportability issues
- Cross-platform certificate bundle creation
- SAN requirement for modern browsers



3. Monitoring & Observability Stack
Technologies: Prometheus, Grafana, Node Exporter  
Skills: Metrics collection, dashboard creation, alerting

- Deployed Prometheus for time-series metrics collection
- Configured exporters for Linux (Node Exporter) and Windows
- Built custom Grafana dashboards for infrastructure monitoring
- Integrated Pi-hole metrics into monitoring

Metrics Tracked:
- CPU, memory, disk usage across all VMs
- Network traffic per VLAN
- DNS query statistics
- Service uptime



4. WireGuard VPN
Technologies: WireGuard, pfSense, NAT  
Skills: VPN configuration, remote access, cryptography

- Configured WireGuard server on pfSense
- Established secure remote access to all VLANs
- Implemented split-tunnel routing
- Configured port forwarding through ISP gateway

Key Achievement: Full homelab access from anywhere via encrypted tunnel

**Troubleshooting:** Resolved NAT traversal issues and pfSense WAN firewall blocking



5. Pi-hole Network-Wide Ad Blocking
Technologies: Pi-hole, DNS, DHCP  
Skills: DNS architecture, ad-blocking, network filtering

- Deployed Pi-hole as primary DNS server
- Configured DNS forwarding from pfSense
- Manages 100,000+ blocked domains
- Integrated with Prometheus for statistics


6. Python Network Scanner (eyespy.py)
Technologies: Python, nmap, JSON, XML parsing  
Skills: Python development, network discovery, automation

Features:
- Automated network scanning across multiple VLANs
- Device discovery (IP, MAC, hostname, open ports)
- Baseline comparison for new device detection
- Alert generation for unauthorized devices
- Persistent storage with JSON

Code Highlights:
- XML parsing for nmap output
- Dictionary-based device tracking
- File I/O for baseline persistence
- Error handling and logging


7. Automated VM Patching (Bash)
Technologies: Bash, SSH, APT  
Skills: Shell scripting, SSH automation, configuration management

Features:
- Passwordless SSH via key-based authentication
- Sequential VM updates with error handling
- Comprehensive logging with timestamps
- Non-interactive mode for unattended execution

3 Linux VMs (Ubuntu Mgt, Monitoring, Pi-hole)



8. Jumpbox Architecture
Technologies: SSH, bastion host security  
Skills: Privileged access management, defense in depth

- Established secure SSH gateway for infrastructure access
- Implemented SSH key-based authentication
- Restricted direct access to production VMs


Skills Demonstrated

Cloud & Virtualization
- Proxmox VE hypervisor management
- VM lifecycle management
- Resource allocation and optimization
- Infrastructure as Code principles

Networking
- VLAN design and implementation
- Firewall rule configuration
- NAT and port forwarding
- VPN tunneling (WireGuard)
- DNS architecture

Security
- PKI and certificate management
- Zero-trust network segmentation
- Secure remote access (VPN, jumpbox)
- Network traffic filtering
- Threat detection preparation

Systems Administration
- Windows Server (AD, DNS, CA)
- Linux administration (Ubuntu, Debian)
- Service management (systemd)
- User and permission management

Automation & Scripting
- Python (network scanning, API integration)
- Bash (system automation, patching)
- SSH automation
- Cron scheduling

Monitoring & Observability
- Prometheus metrics collection
- Grafana dashboard creation
- Log aggregation concepts
- Performance baselining

Technical Skills Acquired
- Designed enterprise network from scratch
- Implemented production-grade security controls
- Developed custom automation tools
- Deployed monitoring infrastructure
- Managed certificate lifecycle

Soft Skills Developed
- Documentation and knowledge transfer
- Systematic troubleshooting methodology
- Project planning and execution
- Resource optimization
- Self-directed learning


LinkedIn: https://www.linkedin.com/in/sam-stockhausen/  
 
Certifications:
- CompTIA Security+
- CompTIA CySA+
- CompTIA CSAP
- Microsoft Azure Fundamentals (AZ-900)
- Certificate of Cloud Security Knowledge (CCSK)
- Google Cybersecurity Professional Certificate
- NSA CAE Cyber Defense
- TryHackMe: SOC Level 1 
