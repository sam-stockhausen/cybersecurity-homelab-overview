# Overview
Deployed OpenCanary as a Docker-based honeypot on Ubuntu-Mgt to detect internal
lateral movement across VLAN 20. Fake SSH, HTTP, RDP, and SMB listeners record
every probe attempt as structured JSON. Logs are shipped via Promtail to Loki and
surfaced in Grafana with a firing alert rule — building the detection pipeline that
Wazuh will consume once RAM allows.

---

## Architecture
Ubuntu-Mgt (192.168.20.20)<br>
├── opencanary container — fake service listeners<br>
│   ├── SSH :2222<br>
│   ├── HTTP :8880<br>
│   ├── RDP :33389<br>
│   └── SMB :445<br>
├── promtail container — tails /var/log/opencanary/opencanary.log<br>
├── loki container — log aggregation backend :3100<br>
└── Grafana (VM 104) — logs panel + alert rule<br>

---

## Implementation

**OpenCanary Deployment**
- Created `~/opencanary/opencanary.conf` with fake service config tuned for VLAN 20
- Set `ip.ignorelist` to exclude 192.168.20.20 (own VM) and 127.0.0.1
- Used `network_mode: host` in Docker Compose so fake ports bind directly to the VM NIC and are reachable by other VMs on VLAN 20
- Mapped config in as read-only volume; logs written to named Docker volume shared with Promtail
- Verified all three services started: HTTP, RDP, SSH (portscan module disabled in Docker — requires host-level network access)
<br>
<img width="631" height="87" alt="ports" src="https://github.com/user-attachments/assets/cb2b0506-9008-48d5-b0a6-6cd5890f9c5a" />
*docker compose logs showing CanaryHTTP, CanaryRDP, and HoneyPotSSHFactory starting on their respective ports*<br>
<br>
**Log Pipeline — Promtail + Loki**
- Deployed Loki and Promtail in the same Docker Compose stack alongside OpenCanary
- Promtail configured to tail `/var/log/opencanary/opencanary.log` from the shared named volume
- JSON pipeline stages extract `logtype`, `src_host`, `dst_port`, and `node_id` as Loki labels for efficient querying
- Confirmed pipeline with `curl http://localhost:3100/loki/api/v1/labels` returning all expected label names<br>
<br>
<img width="928" height="268" alt="tail" src="https://github.com/user-attachments/assets/4fd61648-269a-43ed-9f85-dd3ab1605ad0" />

*curl output showing dst_port, host, job, logtype, node_id, src_host labels confirmed in Loki*<br>
<br>
**Grafana Integration**
- Added Loki as a data source on Grafana (VM 104) at `http://192.168.20.20:3100`
- Created logs panel with LogQL query filtering out startup events (logtype 1001):
```
{job="opencanary"} | json | logtype != `1001`
```
- Created alert rule using `sum by (job)` metric query, Reduce expression (Last), and Threshold > 0
- Alert fires within one evaluation window of any honeypot probe<br>
<br>
<img width="847" height="540" alt="dash" src="https://github.com/user-attachments/assets/7f953ec7-ab73-4058-8e5d-b03e758ea0ec" />
*Grafana logs panel showing SSH connection (logtype 4000) and auth attempt (logtype 4002) with src_host 192.168.20.30*<><br>
<br>
<img width="707" height="218" alt="firing" src="https://github.com/user-attachments/assets/b154ca6a-afdf-4399-b4bc-347ea29ec149" />
*Grafana alert rule in Firing state after test SSH probe from Win10-Client*<br>
<br>
---

## Deployment Workflow

Create config and compose files in `~/opencanary/`<br>
Start stack with `docker compose up -d`<br>
Verify fake services listening with `ss -tlnp`<br>
<br>
<img width="631" height="87" alt="ports" src="https://github.com/user-attachments/assets/61baa912-edf4-4aff-a9a4-960ec4013e2b" />

*ss -tlnp output showing OpenCanary bound on 2222, 8880, and 33389*<br>
<br>
Trigger test probe from Win10-Client or Monitoring VM<br>
Confirm JSON alert written to opencanary.log<br>
<br>
<img width="788" height="567" alt="logs" src="https://github.com/user-attachments/assets/6056a8bb-2c7c-4551-be85-078fb1cf82c4" />

*opencanary.log showing logtype 4000 (connection), 4001 (version exchange), 4002 (auth attempt) sequence with captured username and password*<br>
<br>
Confirm Loki labels populated<br>
Add Loki data source in Grafana, build logs panel<br>
Configure alert rule and verify firing state<br>

---

## Log Types Reference

| Logtype | Description |
|---------|-------------|
| 1001 | Service startup / internal message |
| 4000 | SSH connection opened |
| 4001 | SSH version exchange |
| 4002 | SSH auth attempt — captures username + password |
| 5000 | HTTP request |
| 9000 | RDP connection |

---

## Key Concepts Demonstrated
- Deception-based internal detection using honeypot fake services
- Docker named volume sharing between containers for log pipeline
- Promtail JSON pipeline stages for label extraction
- Loki log aggregation and LogQL querying
- Grafana alert rule construction for log-based metrics — Reduce + Threshold pattern
- Forward-compatible syslog output design for future Wazuh agent ingestion

---

## Challenges

**Promtail connecting to Loki via 127.0.0.1**<br>
Promtail was configured to push to `http://127.0.0.1:3100` which resolves to the
Promtail container itself, not the Loki container. Fixed by updating the client URL
to the host IP `http://192.168.20.20:3100` so traffic routes through the host
network to Loki's published port.

**Grafana alert — frame cannot be uniquely identified**<br>
Initial LogQL metric query returned multiple streams which Grafana could not map to
a single alert frame. Fixed by grouping with `sum by (job)` to collapse all streams
into one uniquely-labelled series.

**Grafana alert — data is time series, only reduced data can be alerted on**<br>
Grafana requires a Reduce expression between the raw metric query and the Threshold
condition. Added a Reduce (B) expression set to Last feeding into the Threshold (C)
condition, with the alert condition set to C.

**Portscan module disabled in Docker**<br>
OpenCanary's portscan detection requires raw socket access unavailable inside Docker.
Noted as a limitation — would require a dedicated VM or host-level deployment to enable.

---

## Tools & Technologies
- OpenCanary (Thinkst) — honeypot framework
- Docker + Docker Compose — container orchestration
- Promtail v3.6.8 — log shipper
- Grafana Loki — log aggregation backend
- Grafana — visualization and alerting
- LogQL — Loki query language
- Python3 json.tool — log formatting and verification
