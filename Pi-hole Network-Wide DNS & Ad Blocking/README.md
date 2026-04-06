# Overview

Deployed Pi-hole as the primary DNS resolver for the entire homelab network. All DNS queries from every VLAN are forwarded through Pi-hole, providing network-wide ad blocking, DNS filtering, and query logging with metrics exported to Prometheus.

---

## Architecture

```
All VMs / Clients
       │
       │  DNS queries
       ▼
   [Pi-hole]  ←── 100,000+ blocklist domains
       │
       │  unblocked queries forwarded upstream
       ▼
  Upstream DNS (e.g. Cloudflare 1.1.1.1 / Google 8.8.8.8)
```

Pi-hole runs on a dedicated lightweight Debian 12 VM in the Servers VLAN.

---

## Implementation

**Pi-hole Deployment**
- Deployed on a minimal Debian 12 VM (1 vCPU, 512MB RAM)
- Installed via the official Pi-hole installer script
- Assigned a static IP on the Servers VLAN
- Configured upstream DNS resolvers

**pfSense DNS Integration**
- Set Pi-hole as the DNS server advertised via DHCP on all VLANs
- Configured pfSense DNS Resolver to forward queries to Pi-hole
- Added a firewall rule to redirect any hardcoded DNS (port 53) to Pi-hole

**Local DNS Records**
- Used Pi-hole's Local DNS Records feature to resolve internal hostnames
- Added A records for all internal services (Grafana, Proxmox, DC01, etc.)
- This allows friendly names (e.g. `grafana.homelab.local`) to resolve internally

**Blocklists**
- Added curated blocklists covering ads, trackers, and malicious domains
- Total blocked domains: 100,000+
- Allowlisted specific domains required for internal tools and Windows Update

**Prometheus Integration**
- Deployed Pi-hole Exporter as a sidecar to expose metrics
- Scraped by Prometheus every 30 seconds
- Grafana dashboard shows query volume, block rate, and top blocked domains

---

## Key Concepts Demonstrated

- DNS architecture and recursive resolution
- Network-wide filtering via DNS sinkholing
- DHCP and DNS integration
- DNS-based threat intelligence (blocklists)
- Metrics export and observability for DNS infrastructure
- Local DNS for internal service discovery

---

## Challenges

- Some Windows Update and Microsoft telemetry domains were blocked by default blocklists — required careful allowlisting to avoid breaking Windows VMs
- Ensuring Pi-hole had high availability (it's a critical service) — startup order configured so Pi-hole starts before other VMs that depend on DNS

---

## Metrics Tracked

| Metric                | Description                        |
|-----------------------|------------------------------------|
| Total queries         | DNS requests per time period       |
| Queries blocked       | Count and percentage blocked       |
| Top blocked domains   | Most frequently sinkholed domains  |
| Top clients           | Which VMs generate the most queries|
| Upstream response time| Latency to upstream resolvers      |

---

## Tools & Technologies

- Pi-hole (Debian 12)
- pfSense DNS Resolver (Unbound)
- Pi-hole Exporter
- Prometheus + Grafana
