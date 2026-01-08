# Active Connection Defense

Techniques for monitoring and terminating suspicious connections during competition.

---

## View Active Connections

### netstat Command

Basic usage (too much output):
```bash
netstat
```

**Useful filtered version:**
```bash
netstat -tu      # TCP and UDP connections only
netstat -tun     # + resolve port numbers (shows 22 instead of "ssh")
netstat -tuna    # + show listening ports too
sudo netstat -tunap  # + show process IDs (requires sudo)
```

**Remember: `netstat -tunap`** (tuna + p)

| Flag | Meaning |
|------|---------|
| `-t` | TCP connections |
| `-u` | UDP connections |
| `-n` | Show port numbers (not names) |
| `-a` | Show all (including listening) |
| `-p` | Show process IDs (requires sudo) |

Example output:
```
Proto  Local Address         Foreign Address       State       PID/Program
tcp    192.168.8.2:22       192.168.8.100:51736   ESTABLISHED 26546/sshd: jenny
tcp    192.168.8.2:22       192.168.8.100:51732   ESTABLISHED 26540/sshd: bob
```

### Filter with grep
```bash
sudo netstat -tunap | grep ESTABLISHED
sudo netstat -tunap | grep ssh
```

### ss Command (alternative to netstat)

Some systems don't have netstat - use `ss` instead:
```bash
ss
ss -t      # TCP only
ss | grep ESTABLISHED
```

---

## View Logged-In Users

### w Command
```bash
w
```

Shows:
- Username
- TTY (terminal)
- From (IP address for remote, `:0` for local GUI)
- Login time
- What they're running

Example output:
```
USER     TTY      FROM             LOGIN@   WHAT
sandbox  :0       :0               09:00    /usr/bin/gnome-shell   <- Local GUI
bob      pts/1    192.168.8.100    10:15    -bash                  <- Remote SSH
jenny    pts/2    192.168.8.100    10:16    -bash                  <- Remote SSH
```

**Note:** `:0` means local GUI session (probably your teammate), IP address means remote connection (possibly attacker).

---

## Kill Connections

### Kill by Process ID

1. Find the PID with `netstat -tunap`
2. Kill it:
```bash
sudo kill <PID>
```

Example:
```bash
sudo netstat -tunap | grep ESTABLISHED
# See jenny's connection has PID 26546
sudo kill 26546
```

### Kill by Username

Log out all sessions for a specific user:
```bash
sudo pkill -kill -u jenny
sudo pkill -kill -u bob
```

**⚠️ WARNING: Don't kill yourself!**
```bash
sudo pkill -kill -u sandbox  # This kills YOUR session too!
```

If attacker is using the same account as you, kill by PID instead.

---

## Monitor Processes

### top Command
```bash
top      # Live view of running processes
htop     # Fancier version (may need to install)
```

Press `q` to quit.

### ps Command
```bash
ps -aux   # Show all processes with details
ps -aux | grep python   # Find Python scripts
ps -aux | grep bash     # Find bash scripts
```

Look for suspicious scripts running in background (attackers may leave these).

---

## Send Messages to Users

### Broadcast to All Users
```bash
wall "Server shutting down in 5 minutes. Please save your work."
```

All logged-in users see the message in their terminal.

### Message Specific User
```bash
w                           # Find their TTY (e.g., pts/2)
sudo write bob pts/2        # Opens interactive message
# Type your message, Ctrl+C to end
```

---

## Competition Strategy

### Active Defense Workflow

1. **Monitor continuously:**
   ```bash
   sudo netstat -tunap | grep ESTABLISHED
   w
   ```

2. **Identify suspicious connections:**
   - Unknown usernames
   - Connections from unexpected IPs
   - Multiple sessions from same IP

3. **Kill suspicious connections:**
   ```bash
   sudo kill <PID>           # By process ID
   sudo pkill -kill -u <user>  # By username
   ```

4. **Meanwhile, teammate secures server:**
   - Change passwords
   - Lock down user accounts
   - Enable firewall
   - Remove unnecessary services

### Watch Out For

- **Background scripts**: Attackers may leave Python/bash scripts running
  ```bash
  ps -aux | grep python
  ps -aux | grep bash
  ```
- **Cron jobs**: Check `crontab -l` and `/etc/cron.*`
- **Friendly fire**: Don't kill your own sessions or teammates!

---

## Quick Reference

| Task | Command |
|------|---------|
| View connections | `sudo netstat -tunap` |
| View logged-in users | `w` |
| Kill by PID | `sudo kill <PID>` |
| Kill by username | `sudo pkill -kill -u <user>` |
| View processes | `ps -aux` or `top` |
| Broadcast message | `wall "message"` |
| Message specific user | `sudo write <user> <tty>` |
