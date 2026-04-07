# Overview

Deployed a Windows Server 2022 Enterprise Root CA and built a full internal PKI to issue SSL certificates for all homelab services. All internal services now serve HTTPS with valid certificates trusted by domain-joined machines — no browser warnings.

---

## Architecture

```
Enterprise Root CA (DC01 - Windows Server 2022)
        │
        ├── Grafana (Ubuntu) — HTTPS on port 3000
        ├── Proxmox — HTTPS on port 8006
        ├── Pihole — HTTPS on port 443
        └── Pfsense - HTTPS on port 443
```

---

## Implementation

**CA Deployment**
- Installed Active Directory Certificate Services (AD CS) role on DC01
- Configured as an Enterprise Root CA tied to the `homelab.local` AD domain
- Set certificate validity periods and key length (RSA 2048-bit minimum)

**Certificate Issuance**
- Created certificate templates with Subject Alternative Names (SAN)
- Issued certificates via the CA web enrollment interface and `certreq`
- Exported certificates in PEM format for use on Linux hosts

**Certificate Distribution**
- Windows machines: auto-enrolled via Group Policy (domain-joined machines trust the CA automatically)
- Linux machines: manually imported the CA root certificate into the system trust store
- Configured each service to load its certificate and private key

**Grafana HTTPS Configuration**
- Generated a CSR on the monitoring VM
- Signed by the internal CA
- Updated `grafana.ini` to reference the cert and key paths
- Restarted Grafana and verified HTTPS

---

## Certificate Workflow (per service)
 
```
1. Generate CSR on target host (OpenSSL or pfSense UI)
2. Submit CSR to DC01 Enterprise CA via certreq
3. Export signed cert from DC01
4. Transfer cert back to target host
5. Install cert per service
6. Install CA root on client machines
```

## Key Concepts Demonstrated

- Public Key Infrastructure (PKI) design
- Certificate Authority deployment and management
- Certificate lifecycle: request → sign → deploy → trust
- Cross-platform certificate distribution (Windows + Linux)
- SAN requirements for modern TLS
- Group Policy for automatic certificate trust

---

## Challenges

- Windows Server defaults to marking private keys as non-exportable — required specific template configuration to allow export for use on Linux
- Assembling the correct certificate chain (cert + intermediate + root) for services that require a full bundle
- Modern browsers require SAN fields; CN-only certificates are rejected
- Proxmox noVNC console has no clipboard support — required HTTP-based file transfer between VMs

---

## Tools & Technologies

- Windows Server 2022 — AD CS role
- OpenSSL — CSR generation and certificate inspection
- Group Policy — CA trust distribution
- certreq / MMC Certificates snap-in
- Python3 http.server / uploadserver — file transfer between VMs
