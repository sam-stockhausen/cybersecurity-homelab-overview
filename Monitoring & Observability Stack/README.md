# Overview

Deployed a full metrics pipeline using Prometheus and Grafana to provide real-time visibility across all homelab infrastructure. Custom dashboards surface CPU, memory, disk, network, and DNS metrics from every VM in the environment.

---

## Architecture

```
[Node Exporter]  [Windows Exporter]  [Pi-hole Exporter]
       │                  │                   │
       └──────────────────┴───────────────────┘
                          │
                    [Prometheus]          ← scrapes all targets
                          │
                      [Grafana]           ← visualizes + alerts
```

All components run on the dedicated Monitoring VM (Ubuntu 24.04, Servers VLAN).

---

## Implementation

**Prometheus**
- Deployed via Docker Compose on the monitoring VM
- Configured `prometheus.yml` with scrape jobs for each target
- Set scrape interval and retention period
- Verified targets via the Prometheus web UI

**Exporters**
- Node Exporter — installed on all Linux VMs via systemd service
- Windows Exporter — installed on DC01 and Win10-Client
- Pi-hole Exporter — runs as a sidecar container, queries the Pi-hole API

**Grafana**
- Deployed via Docker Compose alongside Prometheus
- Configured Prometheus as a data source
- Built custom dashboards for:
  - Per-VM CPU, memory, and disk usage
  - Network traffic per interface
  - DNS query volume and block rate (Pi-hole)
  - Service uptime overview
- Enabled HTTPS using an internal CA-signed certificate

---

## Key Concepts Demonstrated

- Pull-based metrics architecture (Prometheus scraping model)
- Exporter pattern for system and application metrics
- PromQL for querying and aggregating time-series data
- Dashboard design for infrastructure observability
- Docker Compose for multi-container service deployment
- TLS configuration for internal web services

---

## Challenges

- Windows Exporter requires firewall exceptions on the Windows VMs for Prometheus to scrape
- Pi-hole's API changes between versions — required matching the exporter version to the Pi-hole release
- Grafana dashboard panels need careful PromQL to avoid misleading aggregations across multiple instances

---

## Sample PromQL Queries

```promql
# CPU usage per VM
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk usage
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100
```

---

## Tools & Technologies

- Prometheus 2.x
- Grafana 10.x
- Node Exporter (Linux)
- Windows Exporter
- Pi-hole Exporter
- Docker + Docker Compose
