# Overview

Configured a WireGuard VPN server on pfSense to provide secure, encrypted remote access to all homelab VLANs from any location. Implemented split-tunnel routing so only internal traffic traverses the VPN while regular internet traffic exits locally.

---

## Architecture

```
Remote Device (10.10.10.x/24)
        │
        │  WireGuard tunnel (UDP)
        │
[ISP Gateway] → port forward → [pfSense WireGuard Server]
                                         │
                              ┌──────────┼──────────┐
                           VLAN 20    VLAN 30    VLAN 10
                          (Servers)  (Clients)   (Mgmt)
```

---

## Implementation

**pfSense WireGuard Setup**
- Installed the WireGuard package via pfSense package manager
- Generated server keypair on pfSense
- Created a WireGuard tunnel interface with a dedicated VPN subnet
- Assigned the tunnel interface and enabled it in pfSense

**Client Configuration**
- Generated keypair per client device
- Added each client as a peer on the server with an allowed IP from the VPN subnet
- Configured client `wg0.conf` with server endpoint, public key, and allowed IPs

**Routing**
- Added a static route in pfSense for the VPN subnet
- Created firewall rules on the WireGuard interface to permit access to internal VLANs
- Configured split-tunnel: only RFC 1918 ranges route through the VPN

**NAT / Port Forwarding**
- Forwarded the WireGuard UDP port from the ISP gateway to pfSense WAN
- Confirmed inbound UDP was not being blocked by pfSense WAN rules

**DNS over VPN**
- Set the VPN client DNS to point at the internal Pi-hole
- Verified `.homelab.local` names resolve correctly when connected remotely

---

## Key Concepts Demonstrated

- Modern VPN protocol design (WireGuard vs legacy IPsec/OpenVPN)
- Public/private key cryptography for peer authentication
- Split-tunnel vs full-tunnel routing trade-offs
- NAT traversal and port forwarding
- Firewall rule design for VPN traffic
- DNS configuration for remote access

---

## Challenges

- pfSense WAN rules block all inbound by default — required an explicit allow rule for the WireGuard UDP port
- NAT traversal with double-NAT (ISP modem + pfSense) required careful port forward configuration
- Split-tunnel allowed-IP calculation to avoid routing conflicts with the local network

---

## Client Config Template

```ini
[Interface]
PrivateKey = <client-private-key>
Address = 10.10.10.x/24
DNS = <internal-dns-ip>

[Peer]
PublicKey = <server-public-key>
Endpoint = <your-public-ip>:<wireguard-port>
AllowedIPs = 192.168.0.0/16   # route internal ranges only (split tunnel)
PersistentKeepalive = 25
```

---

## Tools & Technologies

- WireGuard (pfSense package)
- pfSense 2.7.x
- NAT / port forwarding
- Split-tunnel routing
