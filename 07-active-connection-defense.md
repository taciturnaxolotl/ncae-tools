# Active Connection Defense

## Overview
Monitoring and managing active network connections is critical during competitions. This guide covers tools for identifying who's connected to your system and how to terminate malicious connections.

## Core Monitoring Tools

### netstat - Network Statistics

**Most useful form**:
```bash
sudo netstat -tunap
```

**Breakdown**:
- `-t` = TCP connections
- `-u` = UDP connections  
- `-n` = Show numeric ports (22 instead of "ssh")
- `-a` = Show listening and established connections
- `-p` = Show process IDs (requires sudo)

**Output columns**:
```
Proto  Local Address      Foreign Address    State       PID/Program
tcp    192.168.195.100:22 192.168.195.2:51736 ESTABLISHED 265408/sshd
```

**Common filters**:
```bash
netstat -tunap | grep ESTABLISHED   # Only active connections
netstat -tunap | grep :22           # Only SSH connections
netstat -tunap | less               # Scroll through output
```

### ss - Socket Statistics

Modern replacement for netstat. Similar syntax:

```bash
ss                          # Basic output (lots of info)
ss | grep ESTAB             # Only established connections
ss -tunap                   # Same flags as netstat
```

**Advantage**: ss is installed on more modern systems by default.

### w - Who is logged in

```bash
w
```

**Shows**:
- Username
- From where (IP address or `:0` for local console)
- Login time
- What they're doing

**Example output**:
```
USER     FROM             WHAT
sandbox  :0               -bash
bob      192.168.195.2    -bash
jenny    192.168.195.2    -bash
```

**Key indicator**:
- `:0` = Local console (physically at the machine)
- IP address = Remote connection (SSH, etc.)

## Finding Process Information

### top - Interactive Process Viewer

```bash
top
```

- Shows CPU/memory usage
- Lists running processes
- Press `q` to quit

### htop - Enhanced Process Viewer

```bash
htop                 # If installed (not always available)
```

More colorful and interactive than `top`.

### ps - Process Status

```bash
ps aux               # All processes, all users
ps aux | grep ssh    # Find SSH processes
```

## Killing Connections

### Kill by Process ID (PID)

1. **Find the PID**:
```bash
sudo netstat -tunap
# Example output shows PID 265465 for jenny's SSH connection
```

2. **Kill the process**:
```bash
sudo kill 265465
```

**From the user's perspective**: Connection closes immediately
```
Connection to 192.168.195.100 closed by remote host.
```

### Kill by Username (pkill)

```bash
sudo pkill -kill -u jenny        # Kill all processes for user jenny
sudo pkill -kill -u bob          # Kill all processes for user bob
```

**Warning**: This kills ALL processes for that user, including:
- Active SSH sessions
- Running programs
- Background jobs

### Kill Signal Types

```bash
sudo kill PID              # SIGTERM (graceful shutdown, default)
sudo kill -9 PID           # SIGKILL (force kill immediately)
sudo pkill -kill -u user   # -kill = SIGKILL
```

## Competition Workflow

### Active Defense Pattern

1. **Someone monitors connections**:
```bash
# Run periodically or in a loop
sudo netstat -tunap
```

2. **Identify suspicious connections**:
- Unknown IP addresses
- Unexpected users logged in
- Unusual ports

3. **Kill immediately**:
```bash
sudo pkill -kill -u <suspicious_user>
# or
sudo kill <PID>
```

4. **Someone else hardens the system**:
- Change passwords
- Disable accounts
- Configure firewall
- Close unnecessary services

### Example Monitoring Script

```bash
#!/bin/bash
# Quick connection checker
while true; do
    clear
    echo "=== Active SSH Connections ==="
    sudo netstat -tunap | grep :22 | grep ESTABLISHED
    sleep 5
done
```

## Common Scenarios

### Scenario 1: Unknown SSH Connection

```bash
# See who's connected
w

# Find their process ID
sudo netstat -tunap | grep ESTABLISHED

# Kill by PID
sudo kill 265465
```

### Scenario 2: Brute Force Attempts

```bash
# See all connection attempts
sudo netstat -tunap | grep :22

# Check auth logs
sudo tail -f /var/log/auth.log

# Block the source IP with firewall
sudo ufw deny from <attacker_ip>
```

### Scenario 3: Multiple Sessions from Same User

```bash
# Kill all sessions for a user
sudo pkill -kill -u jenny

# Disable the account
sudo passwd -l jenny        # Lock password
sudo usermod -s /bin/false jenny    # Disable shell
```

## Warnings and Gotchas

### Don't Kill Yourself

```bash
# BAD - if you're logged in as sandbox:
sudo pkill -kill -u sandbox
# This kills YOUR session too!
```

**Better approach**: Kill by specific PID if you're using the same username.

### Don't Kill Teammates

- Check with team before killing connections
- Look at FROM addresses to identify internal vs external
- Local (`:0`) connections are usually teammates at the console

### Shared Accounts

If red team is using the same account as you:
- Kill by PID (specific to their connection)
- Don't kill by username (you'll disconnect yourself)

## Process Information Fields

**Understanding PID in netstat**:
```bash
sudo netstat -tunap
```

Output:
```
PID/Program name
265408/sshd: sandbox
265465/sshd: jenny
```

- PID: Process ID (unique number)
- Program: Which service (sshd, apache2, etc.)
- User context: Which user owns the process

## Monitoring vs. Hardening

**Active monitoring** (short-term):
- Running netstat/ss repeatedly
- Killing suspicious connections as they appear
- Playing "whack-a-mole"

**Hardening** (long-term):
- Change passwords
- Disable unused accounts
- Configure firewall rules
- Close unnecessary services
- Update vulnerable software

**Best practice**: Use monitoring to buy time while someone else hardens the system. You can't watch connections for 6 hours straight.

## Tool Availability

| Tool | Typical Availability |
|------|---------------------|
| netstat | Most systems (may need `net-tools` package) |
| ss | Modern systems (usually pre-installed) |
| w | All Unix/Linux systems |
| top | All Unix/Linux systems |
| htop | Optional (install with apt/yum) |
| ps | All Unix/Linux systems |

**If netstat is missing**:
```bash
sudo apt install net-tools       # Debian/Ubuntu
sudo yum install net-tools       # CentOS/RHEL
```

Or just use `ss` instead.
