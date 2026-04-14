# Overview

Deployed Nginx Proxy Manager (NPM) via Docker on `Ubuntu-Mgt` as a reverse proxy for all internal homelab services. All traffic is routed through NPM using a wildcard SSL certificate issued by the internal Enterprise CA, providing centralized HTTPS termination across VLANs.

## Environment

| Host | IP | Role |
|---|---|---|
| Ubuntu-Mgt (VM 101) | 192.168.20.20 | Docker host, NPM |
| DC01 (VM 102) | 192.168.20.10 | Enterprise CA, cert issuance |
| Pi-hole (VM 105) | 192.168.20.50 | DNS — updated to point to NPM |

## Architecture

```
Client → Pi-hole DNS (*.homelab.local → 192.168.20.20)
       → NPM (192.168.20.20)
       → Backend service (Proxmox / pfSense / Grafana / Pi-hole)
```

All hostnames resolve to NPM's IP. NPM terminates TLS with the wildcard cert and forwards to each backend.

---

## Steps

### 1. Install Docker on Ubuntu-Mgt

```bash
sudo apt update && sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Deploy NPM via Docker Compose

```bash
sudo mkdir -p /opt/npm && cd /opt/npm

sudo tee /opt/npm/docker-compose.yml << 'EOF'
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
EOF

docker compose up -d
docker compose ps
```

<img width="546" height="56" alt="dockerversion" src="https://github.com/user-attachments/assets/5fc5f4fe-6930-493a-8ae7-1045b93b537f" />

Access the admin UI at `http://192.168.20.20:81`.  
Default credentials: `admin@example.com` / `changeme` — forced password change on first login.

<img width="830" height="318" alt="NPM" src="https://github.com/user-attachments/assets/07865e30-8c44-4791-8690-03bf38904094" />


---

### 3. Update Pi-hole DNS

All `*.homelab.local` records updated to point to NPM (`192.168.20.20`) so traffic routes through the proxy.

**Local DNS → DNS Records:**

| Domain | IP |
|---|---|
| proxmox.homelab.local | 192.168.20.20 |
| pfsense.homelab.local | 192.168.20.20 |
| grafana.homelab.local | 192.168.20.20 |
| pihole.homelab.local | 192.168.20.20 |
| npm.homelab.local | 192.168.20.20 |

---

### 4. Issue Wildcard Certificate from Enterprise CA

On DC01, a custom certificate template (`WebServerCustom`) was created by duplicating the built-in Web Server template with **Supply in request** enabled for the Subject Name. The template was added to the CA's issuable templates via `certsrv.msc`.

```powershell
# Generate CSR
$inf = @"
[Version]
Signature="`$Windows NT`$"

[NewRequest]
Subject = "CN=*.homelab.local"
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = TRUE
RequestType = PKCS10
"@

$inf | Out-File C:\wildcard.inf -Encoding ASCII
certreq -new C:\wildcard.inf C:\wildcard.csr

# Submit to CA
certreq -submit -attrib "CertificateTemplate:WebServerCustom" -config "DC01\homelab-ca" C:\wildcard.csr C:\wildcard.cer

# Accept and install
certreq -accept C:\wildcard.cer

# Export to PFX
Export-PfxCertificate -Cert "Cert:\LocalMachine\My\<thumbprint>" -FilePath C:\wildcard.pfx -Password (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force)
```

Transfer PFX to Ubuntu-Mgt via `python -m http.server 8080` on DC01.

---

### 5. Convert Certificate for NPM

OpenSSL 3.x outputs PKCS#8 keys by default — NPM requires legacy RSA format.

```bash
cd /opt/npm

# Extract clean PEM cert
sudo bash -c 'openssl pkcs12 -in wildcard.pfx -clcerts -nokeys -nomacver -password pass:TempPass123! | openssl x509 -out clean.crt'

# Extract CA chain
sudo bash -c 'openssl pkcs12 -in wildcard.pfx -cacerts -nokeys -nomacver -password pass:TempPass123! | openssl x509 -out clean-ca.crt'

# Extract private key in legacy RSA format (required by NPM)
sudo bash -c 'openssl pkcs12 -in wildcard.pfx -nocerts -nodes -nomacver -password pass:TempPass123! -legacy | openssl rsa -traditional -out wildcard-legacy.key'

sudo chmod 644 clean.crt clean-ca.crt wildcard-legacy.key
```

> **Note:** The `-legacy` flag and `-traditional` output are required on OpenSSL 3.x to produce `BEGIN RSA PRIVATE KEY` format. Without this, NPM rejects the key.

---

### 6. Import Certificate into NPM

Served files via `python3 -m http.server 8080` on Ubuntu-Mgt, downloaded to Win10 client.

In NPM: **SSL Certificates → Add Certificate → Custom**

| Field | File |
|---|---|
| Certificate | `clean.crt` |
| Certificate Key | `wildcard-legacy.key` |
| Intermediate Certificate | `clean-ca.crt` |

Named the cert `wildcard`.

<img width="918" height="170" alt="wildcard" src="https://github.com/user-attachments/assets/2a0d889f-9795-476e-9673-7a65514dd6e5" />


---

### 7. Configure Proxy Hosts

| Domain | Scheme | Forward Host | Port | Websockets | SSL |
|---|---|---|---|---|---|
| proxmox.homelab.local | https | 192.168.0.200 | 8006 | ✅ | wildcard |
| pfsense.homelab.local | https | 192.168.0.100 | 443 | ❌ | wildcard |
| grafana.homelab.local | http | 192.168.20.30 | 3000 | ✅ | wildcard |
| pihole.homelab.local | https | 192.168.20.50 | 443 | ❌ | wildcard |
| npm.homelab.local | http | 192.168.20.20 | 81 | ❌ | wildcard |

> **Note:** Proxmox requires Websockets Support enabled or the console will not function.  
> **Note:** pfSense and Pi-hole use `https` as the forward scheme since they only listen on HTTPS.

<img width="842" height="499" alt="NPM2" src="https://github.com/user-attachments/assets/44b13ac2-14f4-4f51-b408-221621bb8e69" />


---

### 8. Verify

All services confirmed accessible via HTTPS with valid wildcard cert from the internal Enterprise CA — no browser warnings.

<img width="370" height="127" alt="padlock" src="https://github.com/user-attachments/assets/80d77c66-7109-4ccb-9abc-66e6d200ceb8" />


---

## Outcome

- Single point of TLS termination for all internal services
- Wildcard cert eliminates per-service cert management going forward
- NPM admin UI accessible at `https://npm.homelab.local`
- Integrates with Lab 02 (PKI), Lab 05 (Pi-hole DNS), and Lab 03 (AD/CA)

## Services

| Service | URL |
|---|---|
| Proxmox | https://proxmox.homelab.local |
| pfSense | https://pfsense.homelab.local |
| Grafana | https://grafana.homelab.local |
| Pi-hole | https://pihole.homelab.local |
| NPM Admin | https://npm.homelab.local |
