# DNS, Rsync, and Cron Services

## DNS Service (BIND)

### Service Name
- `named` (most distributions)

### Configuration Location
- Ubuntu: `/etc/bind/`
- CentOS: May be in different location

### Check Service
```bash
systemctl status named
```

### Basic Concept
DNS translates domain names to IP addresses (forward lookup) and IP addresses to domain names (reverse lookup).

**Forward lookup**: `example.com` → `192.168.1.100`
**Reverse lookup**: `192.168.1.100` → `example.com`

### Key Files (Bind)
- `named.conf` - Main configuration
- Zone files - Define DNS records for domains

**This is a complex service** - requires understanding of:
- Zone files
- DNS record types (A, PTR, CNAME, MX, etc.)
- Forward vs reverse zones
- DNS hierarchy

---

## Rsync - File Synchronization/Backup

### Basic Syntax
```bash
rsync [options] source destination
```

### Common Options
```bash
-a    # Archive mode (preserves permissions, timestamps, etc.)
-v    # Verbose (show what's being copied)
-z    # Compress during transfer
-r    # Recursive (copy directories)
-h    # Human-readable output
--delete  # Delete files in dest that don't exist in source
```

### Local Backup Example
```bash
rsync -av /home/user/stuff/ /home/user/backups/
```

**Note the trailing slash** on source - affects behavior:
- `/source/` - copy contents of source
- `/source` - copy source directory itself

### Remote Backup via SSH
```bash
rsync -avz /local/path/ user@remote:/remote/path/
```

### Consistency vs. Accumulation

**Consistency** (mirror - deletes old files):
```bash
rsync -av --delete /source/ /backup/
```

**Accumulation** (keeps all files):
```bash
rsync -av /source/ /backup/
```

### Check Installed
```bash
rsync --version
# or just run rsync to see options
```

---

## Cron - Task Automation

### Service Name
- `cron` (Ubuntu/Debian)
- `crond` (CentOS/RHEL)

### Check Service
```bash
systemctl status cron
systemctl status crond  # CentOS
```

### Edit Crontab
```bash
crontab -e   # Edit current user's crontab
```

First time will ask which editor (nano recommended for beginners).

### Crontab Syntax

Five time fields + command:
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-7, 0/7 = Sunday)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

**Asterisk (*) means "every"**

### Examples

Every minute:
```bash
* * * * * /path/to/command
```

Every 5 minutes:
```bash
*/5 * * * * /path/to/command
```

Every day at 2:30 AM:
```bash
30 2 * * * /path/to/command
```

Every Monday at 5:00 PM:
```bash
0 17 * * 1 /path/to/command
```

First day of every month at midnight:
```bash
0 0 1 * * /path/to/command
```

### Automated Backup Example

Run rsync backup every night at 2 AM:
```bash
0 2 * * * rsync -av --delete /var/www/html/ /backups/website/
```

### Redirect Output

Send output to file:
```bash
* * * * * /path/to/command > /path/to/logfile.txt
```

Append to file:
```bash
* * * * * /path/to/command >> /path/to/logfile.txt
```

Suppress output:
```bash
* * * * * /path/to/command > /dev/null 2>&1
```

### View Crontab
```bash
crontab -l   # List current user's crontab
```

### Remove Crontab
```bash
crontab -r   # Remove current user's crontab
```

### System-Wide Cron

User-specific: Managed via `crontab -e`

System-wide cron directories:
- `/etc/cron.daily/` - Scripts run daily
- `/etc/cron.hourly/` - Scripts run hourly
- `/etc/cron.weekly/` - Scripts run weekly
- `/etc/cron.monthly/` - Scripts run monthly

Place executable scripts in these directories for automatic execution.

### Important Notes

1. Cron uses absolute paths - always specify full path to commands
2. Cron runs in minimal environment - may need to set PATH, etc.
3. Test commands manually first before adding to cron
4. Cron jobs run as the user who owns the crontab
5. `sudo crontab -e` edits root's crontab (for privileged tasks)

### Combining Rsync + Cron

Automated nightly backups:
```bash
# In crontab -e:
0 2 * * * rsync -avz /var/www/html/ /backups/website/
0 3 * * * rsync -avz /etc/ /backups/configs/
```

This creates automated, scheduled backups without manual intervention.
