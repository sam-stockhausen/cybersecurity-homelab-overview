# Overview

Implemented a jumpbox (bastion host) architecture using the Ubuntu management VM as a single, hardened SSH gateway into the homelab. No production VMs accept direct SSH connections from outside their VLAN — all access is proxied through the jumpbox, creating a controlled chokepoint for privileged access.

---

## Architecture

```
External / WireGuard VPN
        │
        │  SSH (port 22)
        ▼
  [Ubuntu-Mgt]  ←── Jumpbox / Bastion Host
  (Servers VLAN)
        │
        ├── SSH → pfSense (limited)
        ├── SSH → Monitoring VM
        └── SSH → Pi-hole VM
```

Direct SSH from the client VLAN or VPN to production VMs is blocked at the firewall level. All admin sessions must originate from the jumpbox.

---

## Implementation

**Jumpbox Hardening**
- Key-based SSH authentication only — password authentication disabled
- Root login disabled (`PermitRootLogin no`)
- SSH access restricted to the management user
- `AllowUsers` directive limits which accounts can log in via SSH
- Inactive session timeout configured (`ClientAliveInterval`)

**pfSense Firewall Rules**
- SSH (port 22) inbound allowed only from the jumpbox IP on each server VLAN rule
- All other direct SSH attempts dropped
- WireGuard VPN clients can reach the jumpbox but not production VMs directly

**SSH Agent Forwarding**
- SSH agent forwarding enabled so keys don't need to be stored on the jumpbox itself
- Operators authenticate once to the jumpbox; their local key is used for onward hops

**ProxyJump Configuration**
- Client `~/.ssh/config` uses `ProxyJump` to transparently tunnel through the jumpbox:

```
Host monitoring
    HostName 10.0.20.30
    User admin
    ProxyJump jumpbox

Host jumpbox
    HostName 10.0.20.20
    User admin
    IdentityFile ~/.ssh/id_ed25519
```

With this config, `ssh monitoring` automatically tunnels through the jumpbox in a single command.

---

## Key Concepts Demonstrated

- Bastion host / jump server design pattern
- Privileged access management (PAM)
- SSH hardening best practices
- Defense in depth — multiple layers before reaching production
- Firewall rule design for access control
- SSH ProxyJump for transparent multi-hop access
- Agent forwarding vs key storage trade-offs

---

## Challenges

- Balancing convenience (transparent ProxyJump) with security (not storing keys on the jumpbox)
- Ensuring the WireGuard VPN DNS resolves jumpbox hostname correctly for remote access
- Keeping the jumpbox itself patched and minimal — it's the highest-value target in the network

---

## Tools & Technologies

- OpenSSH (server + client)
- pfSense firewall rules
- SSH key management (ed25519)
- SSH ProxyJump / agent forwarding
