eyespy.py - Network Device Discovery Scanner

Overview
Automated network scanner that discovers devices across multiple VLANs, tracks baseline inventory, and alerts on new or unauthorized devices.

Features
- Multi-VLAN scanning (SERVERS, CLIENTS)
- Device fingerprinting (IP, MAC, hostname, open ports)
- Baseline comparison for change detection
- JSON persistent storage
- New device alerting

Technical Implementation

Technologies
- Python 3: Core language
- nmap: Network scanning engine
- XML parsing: Processing nmap output
- JSON: Data persistence

Architecture
1. Load previous baseline (JSON file)
2. Scan configured networks (nmap -F)
3. Parse XML output → extract device data
4. Compare current vs baseline
5. Alert on new devices
6. Save updated baseline

Key Code Patterns

XML Parsing:
```python
root = ET.fromstring(nmap_output)
for host in root.findall('host'):
    # Extract IP, MAC, hostname, ports
```

Device Dictionary:
```python
devices[ip] = {
    'mac': mac_address,
    'hostname': hostname,
    'ports': [22, 80, 443],
    'first_seen': timestamp,
    'last_seen': timestamp
}
```

Usage
```bash
sudo python3 eyespy.py
```

First run: Establishes baseline
Subsequent runs: Compares and alerts on changes

Configuration

Edit `NETWORKS` list in script:
```python
NETWORKS = [
    "192.168.20.0/24",  # SERVERS
    "192.168.30.0/24",  # CLIENTS
]
```

Baseline location: `~/eyespy/baseline_devices.json`


Output Example:
Scanning 192.168.20.0/24...
Scanning 192.168.30.0/24...
NEW DEVICES DETECTED!
IP: 192.168.20.110
MAC: AA:BB:CC:DD:EE:FF
Hostname: unknown-device
Open Ports: [22, 445]
Baseline saved to /home/admin/eyespy/baseline_devices.json
Scan complete! Found 7 devices.
