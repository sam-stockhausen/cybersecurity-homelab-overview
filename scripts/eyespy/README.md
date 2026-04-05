# eyespy.py — Network Device Discovery Scanner

Automated network scanner that discovers devices across multiple network segments, tracks a baseline inventory, and alerts on new or unauthorized devices.

---

## Features

- Multi-subnet scanning across configurable network ranges
- Device fingerprinting (IP, MAC, hostname, open ports)
- Baseline comparison for change detection
- JSON persistent storage
- New device alerting

---

## Technical Implementation

**Technologies**
- Python 3 — core language
- nmap — network scanning engine
- xml.etree.ElementTree — parsing nmap output
- json — data persistence

**Architecture**
1. Load previous baseline from JSON file
2. Scan configured networks using nmap
3. Parse XML output and extract device data
4. Compare current scan vs baseline
5. Alert on new or changed devices
6. Save updated baseline

---

## Key Code Patterns

**XML Parsing:**
```python
root = ET.fromstring(nmap_output)
for host in root.findall('host'):
    # Extract IP, MAC, hostname, ports
```

**Device Dictionary:**
```python
devices[ip] = {
    'mac': mac_address,
    'hostname': hostname,
    'ports': [22, 80, 443],
    'first_seen': timestamp,
    'last_seen': timestamp
}
```

---

## Configuration

Edit the `NETWORKS` list at the top of the script to match your environment:

```python
NETWORKS = [
    "10.0.20.0/24",   # Example: servers subnet
    "10.0.30.0/24",   # Example: clients subnet
]
```

Baseline file location is also configurable:

```python
BASELINE_FILE = os.path.expanduser("~/eyespy/baseline_devices.json")
```

---

## Usage

```bash
sudo python3 eyespy.py
```

- **First run:** Establishes a device baseline
- **Subsequent runs:** Compares against baseline and alerts on new devices

---

## Output Example

```
Scanning 10.0.20.0/24...
Scanning 10.0.30.0/24...

NEW DEVICES DETECTED!
  IP:         10.0.20.110
  MAC:        AA:BB:CC:DD:EE:FF
  Hostname:   unknown-device
  Open Ports: [22, 445]

Baseline saved to ~/eyespy/baseline_devices.json
Scan complete! Found 7 devices.
```

---

## Requirements

```bash
pip install python-nmap
sudo apt install nmap   # or brew install nmap on macOS
```

nmap must be run with sudo/root privileges for MAC address resolution.
