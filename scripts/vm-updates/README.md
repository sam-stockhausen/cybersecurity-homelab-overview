update-vms.sh - Automated Linux VM Patching

Overview----------------
Bash script for automated patch management across multiple Linux VMs with logging, error handling, and sequential updates.

Features----------------
- Passwordless SSH (key-based authentication)
- Sequential VM updates (fail-safe)
- Comprehensive logging with timestamps
- Non-interactive mode (no prompts)
- Success/failure tracking per VM

Technical Implementation

Technologies-------------

- Bash: Shell scripting

- SSH: Remote execution

- APT: Package management

Architecture--------------
1. Update local management VM
2. Loop through remote VMs:

    -SSH to VM

   -Run apt update && upgrade

    -Log output

    -Track success/failure
   
4. Generate summary report

Key Patterns---------------

VM Configuration Array:
```bash
VMS=(
    "monitoring:192.168.20.30:admin"
    "pihole:192.168.20.50:root"
)
```

String Parsing:
```bash
IFS=':' read -r hostname ip user <<< "$vm_info"
```

Non-Interactive Updates:
```bash
DEBIAN_FRONTEND=noninteractive apt upgrade -y \
    -o Dpkg::Options::="--force-confold"
```

Prerequisites----------------

SSH Key Setup
```bash
# Generate key
ssh-keygen -t ed25519

# Copy to each VM
ssh-copy-id admin@192.168.20.30
ssh-copy-id root@192.168.20.50
```

Sudo Configuration (Optional)
For completely unattended execution:
```bash
# /etc/sudoers.d/apt-nopasswd
admin ALL=(ALL) NOPASSWD: /usr/bin/apt update, /usr/bin/apt upgrade
```

Usage------------------
```bash
~/scripts/update-vms.sh
```

Logs: `~/scripts/logs/vm-updates.log`

Output Example

========================================

VM Update Started: Mon Mar 10 14:30:00 PST 2026

Updating local machine (ubuntu-mgt)...

ubuntu-mgt: SUCCESS

Updating monitoring (192.168.20.30)...

monitoring: SUCCESS

Updating pihole (192.168.20.50)...

pihole: SUCCESS

========================================

VM Update Completed: Mon Mar 10 14:45:23 PST 2026

Configuration---------------

Add/Remove VMs:
Edit `VMS` array:
```bash
VMS=(
    "hostname:ip:username"
)
```

Change Log Location:
```bash
LOG_FILE="/var/log/vm-updates.log"
```
