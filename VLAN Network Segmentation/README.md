# Overview

Designed and implemented a 4-tier VLAN architecture using pfSense and Proxmox to create isolated network segments that mirror enterprise security zone design. Each VLAN enforces explicit access controls with default-deny between segments.

## Architecture

| VLAN | Name    |        Purpose                             |
|------|---------|--------------------------------------------|
| 10   | MGMT    | Reserved for future management hosts       |
| 20   | SERVERS | Internal services and infrastructure       |
| 30   | CLIENTS | End-user workstations                      |
| 99   | DMZ     | Reserved for future public-facing services |





## Implementation

**Proxmox**

Created a VLAN-aware Linux bridge on the Proxmox host
Tagged each VM's virtual NIC to its appropriate VLAN
Passed all VLAN traffic to pfSense via a single trunk interface

**pfSense**

Created VLAN sub-interfaces on the trunk-facing NIC

Assigned each VLAN its own interface with a static gateway IP

Configured DHCP scopes per VLAN

Built firewall rules to control inter-VLAN traffic

**Firewall Rule Design**

Servers VLAN: accepts inbound from Management only by default

Clients VLAN: outbound internet allowed, no direct access to Servers

DMZ: isolated, no inbound access from internal VLANs

All inter-VLAN traffic is statefully inspected


## Key Concepts Demonstrated

Network segmentation and security zone design

Trunk vs access port configuration

Stateful firewall rule logic (default deny, explicit allow)

DHCP scope management per segment

Defense-in-depth through isolation


## Challenges

Configuring the VLAN-aware bridge in Proxmox without losing host connectivity during setup

Ordering pfSense firewall rules correctly (rules are evaluated top-down)

Ensuring DHCP was only serving the correct segment on each interface


## Tools & Technologies

Proxmox VE 8.x
pfSense 2.7.x (FreeBSD)
