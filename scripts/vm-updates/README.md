# update-vms.sh — Automated Linux VM Patching

Bash script for automated patch management across multiple Linux VMs with logging, error handling, and sequential updates.

---

## Features

- Passwordless SSH via key-based authentication
- Sequential VM updates with per-host error handling
- Comprehensive logging with timestamps
- Non-interactive mode for unattended execution
- Success/failure summary report

---

## Technical Implementation

**Technologies**
- Bash — shell scripting
- SSH — remote command execution
- APT — Debian/Ubuntu package management

**Architecture**
1. Update the local management VM first
2. Loop through each remote VM in the configured list
3. SSH to host, run apt update + upgrade
4. Log output and track success/failure per host
5. Generate a summary report at completion

---

## Configuration

Edit the `VMS` array at the top of the script. Each entry follows the format `"hostname:ip:username"`:

```bash
VMS=(
    "monitoring:10.0.20.30:admin"
    "pihole:10.0.20.50:admin"
)
```

Change the log file location if needed:

```bash
LOG_FILE="$HOME/scripts/logs/vm-updates.log"
```

---

## Key Patterns

**VM Configuration Array:**
```bash
VMS=(
    "hostname:ip_address:username"
)
```

**String Parsing:**
```bash
IFS=':' read -r hostname ip user <<< "$vm_info"
```

**Non-Interactive Updates:**
```bash
DEBIAN_FRONTEND=noninteractive apt upgrade -y \
    -o Dpkg::Options::="--force-confold"
```

---

## Prerequisites

**SSH Key Setup**

Generate a key pair on your management host and copy the public key to each target VM:

```bash
ssh-keygen -t ed25519 -C "vm-updater"
ssh-copy-id user@<vm-ip>
```

**Optional: Passwordless sudo for APT**

For fully unattended execution without a password prompt:

```bash
# /etc/sudoers.d/apt-nopasswd
your-user ALL=(ALL) NOPASSWD: /usr/bin/apt update, /usr/bin/apt upgrade
```

---

## Usage

```bash
chmod +x update-vms.sh
~/scripts/update-vms.sh
```

Logs are written to `~/scripts/logs/vm-updates.log` by default.

---

## Output Example

```
========================================
VM Update Started: Mon Mar 10 14:30:00 PST 2026
Updating local machine...
  local: SUCCESS
Updating monitoring...
  monitoring: SUCCESS
Updating pihole...
  pihole: SUCCESS
========================================
VM Update Completed: Mon Mar 10 14:45:23 PST 2026
Results: 3 succeeded, 0 failed
```

---

## Adding or Removing VMs

Edit the `VMS` array — one entry per VM:

```bash
VMS=(
    "hostname1:ip_address:username"
    "hostname2:ip_address:username"
)
```

Remove an entry to exclude a VM from patching.
