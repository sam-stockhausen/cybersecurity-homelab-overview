# Lab 09 — Nginx Proxy Manager

## Overview

Deployed Nginx Proxy Manager (NPM) via Docker on the management VM as a reverse proxy for all internal homelab services. All traffic is routed through NPM using a wildcard SSL certificate issued by the internal Enterprise CA, providing centralized HTTPS termination across VLANs.

---

## Architecture

```
Client → Pi-hole DNS (*.homelab.local → NPM)
       → Nginx Proxy Manager
       → Backend service (Proxmox / pfSense / Grafana / Pi-hole)
```

All internal hostnames resolve to NPM. NPM terminates TLS with the wildcard cert and forwards to each backend service on its native port and protocol.

---

## Implementation

**Docker Deployment**
- Installed Docker Engine and Docker Compose plugin on the management VM
- Deployed NPM as a Docker Compose service with persistent volumes for data and certificates
- NPM listens on ports 80, 443, and 81 (admin UI)

**DNS Update**
- Updated Pi-hole local DNS records so all `*.homelab.local` hostnames resolve to the NPM host
- Previously records pointed directly to each service — rerouting through NPM enables centralized proxy and TLS termination

**Wildcard Certificate**
- Created a custom certificate template (`WebServerCustom`) on the Enterprise CA by duplicating the built-in Web Server template
- Key change required: set Subject Name to **Supply in request** to allow wildcard CNs
- Added the template to the CA's issuable templates via `certsrv.msc`
- Generated a CSR, submitted via `certreq` using the `-attrib` flag to specify the template, and accepted the signed cert
- Exported the certificate chain as a PFX and transferred to the management VM via HTTP file server

**Certificate Conversion**
- OpenSSL 3.x outputs PKCS#8 format by default — NPM requires legacy RSA format (`BEGIN RSA PRIVATE KEY`)
- Used the `-legacy` and `-traditional` flags to extract a compatible private key from the PFX
- Extracted leaf cert and CA chain as separate clean PEM files

**Proxy Hosts**
- Configured a proxy host in NPM for each internal service
- Applied the wildcard cert to all hosts via the SSL tab
- Proxmox required WebSockets Support enabled for console functionality
- pfSense and Pi-hole use HTTPS as the forward scheme since they only serve HTTPS on their backends

---

## Certificate Workflow

```
1. Create custom CA template with "Supply in request" subject name
2. Generate CSR with wildcard CN (*.homelab.local)
3. Submit CSR to Enterprise CA via certreq -attrib flag
4. Accept and export signed cert + chain as PFX
5. Transfer PFX to management VM
6. Convert to PEM format (OpenSSL -legacy flag required for NPM compatibility)
7. Import into NPM as custom certificate
8. Apply to all proxy hosts
```

---

## Proxy Host Summary

| Service | Forward Scheme | Port | Notes |
|---|---|---|---|
| Proxmox | HTTPS | 8006 | WebSockets enabled |
| pfSense | HTTPS | 443 | |
| Grafana | HTTP | 3000 | |
| Pi-hole | HTTPS | 443 | |
| NPM Admin | HTTP | 81 | |

---

## Key Concepts Demonstrated

- Reverse proxy architecture and centralized TLS termination
- Docker and Docker Compose service deployment
- Wildcard certificate issuance from an internal Enterprise CA
- Custom CA template configuration for non-standard certificate subjects
- OpenSSL PFX to PEM conversion and format compatibility across tools
- DNS-based traffic routing through a proxy layer

---

## Challenges

- Enterprise CA required a custom template with **Supply in request** configured — default templates rejected wildcard subject names
- `certreq` requires the `-attrib` flag to specify a template when the INF-based `Template =` directive is not accepted by the CA
- OpenSSL 3.x outputs PKCS#8 private keys by default; NPM only accepts legacy RSA format — resolved with `-legacy` and `-traditional` flags
- NPM file upload required correct PEM formatting — Windows-exported PFX contained bag attributes that had to be stripped

---

## Tools & Technologies

- Docker Engine + Docker Compose
- Nginx Proxy Manager (jc21/nginx-proxy-manager)
- OpenSSL — PFX conversion, key format handling
- Windows Server AD CS — custom template creation, cert issuance
- certreq — CSR submission with template attribute
- Pi-hole — local DNS record management
- Python3 http.server — file transfer between VMs
