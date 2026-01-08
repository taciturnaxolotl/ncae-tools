# DNS Service (BIND)

## Overview

DNS translates domain names to IP addresses (forward lookup) and IP addresses to domain names (reverse lookup).

- **Forward lookup**: `ncaecybergames.org` → `192.168.8.2`
- **Reverse lookup**: `192.168.8.2` → `ncaecybergames.org`

---

## Service Name

- `named` (not `bind`)

```bash
systemctl status named
sudo systemctl start named
sudo systemctl enable named
```

---

## Configuration Locations

### Ubuntu
```
/etc/bind/
├── named.conf                 # Main config (includes other files)
├── named.conf.options         # Server options
├── named.conf.local           # Local zone definitions
├── named.conf.default-zones   # Default zones (localhost, etc.)
├── db.empty                   # Template file to copy for new zones
├── db.local                   # Localhost zone file
├── db.127                     # Localhost reverse zone
└── zones/                     # Your custom zone files (create this)
```

### CentOS/RHEL
Configuration may be in a different location - check `/etc/named/` or `/var/named/`

---

## Key Concepts

### Zone Files
- **Forward zone**: Maps domain names → IP addresses (A records)
- **Reverse zone**: Maps IP addresses → domain names (PTR records)

### Include Structure
The main `named.conf` typically includes other config files:
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

### allow-update Directive

Controls whether dynamic DNS updates are permitted for a zone:

```
zone "example.org" IN {
    type master;
    file "/etc/bind/zones/forward.example.org";
    allow-update { none; };   # No dynamic updates allowed
};
```

| Value | Meaning |
|-------|---------|
| `{ none; }` | No updates allowed (static zone, manual edits only) |
| `{ key mykey; }` | Allow updates signed with a specific TSIG key |
| `{ 192.168.8.5; }` | Allow updates from a specific IP address |

**For competition: Always use `{ none; };`** - prevents attackers from remotely modifying your DNS records.

---

## Setting Up DNS Zones

### Step 1: Add Zone Definitions

Edit the zones config file (Ubuntu: `named.conf.default-zones`):
```bash
sudo nano /etc/bind/named.conf.default-zones
```

Add a **forward lookup zone**:
```
zone "ncaecybergames.org" IN {
    type master;
    file "/etc/bind/zones/forward.ncaecybergames.org";
    allow-update { none; };
};
```

Add a **reverse lookup zone**:
```
zone "8.168.192.in-addr.arpa" IN {
    type master;
    file "/etc/bind/zones/reverse.ncaecybergames.org";
    allow-update { none; };
};
```

**Important**: For reverse zones, write the network portion of the IP **backwards**:
- IP: `192.168.8.x` → Zone: `8.168.192.in-addr.arpa`

---

### Step 2: Create Zone Files Directory

```bash
sudo mkdir /etc/bind/zones
```

---

### Step 3: Copy Template Files

Copy the empty template (preserves correct ownership and permissions):
```bash
sudo cp /etc/bind/db.empty /etc/bind/zones/forward.ncaecybergames.org
sudo cp /etc/bind/db.empty /etc/bind/zones/reverse.ncaecybergames.org
```

**Why copy instead of create from scratch?**
- Preserves correct ownership (`root:bind`)
- Preserves correct permissions (`644`)
- If permissions/ownership are wrong, bind can't read the files

Check permissions:
```bash
ls -l /etc/bind/zones/
```

---

### Step 4: Configure Forward Zone File

```bash
sudo nano /etc/bind/zones/forward.ncaecybergames.org
```

Example forward zone file:
```
$TTL    604800
@       IN      SOA     ncaecybergames.org. root. (
                              2         ; Serial (INCREMENT THIS ON EVERY CHANGE!)
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      sandbox-ubuntu.
sandbox-ubuntu  IN      A       192.168.8.2
www             IN      A       192.168.8.2
```

**Key points:**
- Replace `localhost` with your domain (`ncaecybergames.org.`)
- Replace `localhost` after `NS` with your server hostname (`sandbox-ubuntu.`)
- **Always increment the serial number** when making changes (1→2→3...)
- Add A records for each subdomain

---

### Step 5: Configure Reverse Zone File

```bash
sudo nano /etc/bind/zones/reverse.ncaecybergames.org
```

Example reverse zone file:
```
$TTL    604800
@       IN      SOA     ncaecybergames.org. root.ncaecybergames.org. (
                              2         ; Serial (INCREMENT THIS ON EVERY CHANGE!)
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      sandbox-ubuntu.
2       IN      PTR     www.ncaecybergames.org.
2       IN      PTR     sandbox-ubuntu.ncaecybergames.org.
```

**Key points:**
- For reverse zones, domain names end with a **period** (`.`)
- PTR record uses only the **host portion** of IP (for `192.168.8.2`, just use `2`)
- Multiple PTR records can point to the same IP

---

## Start and Test DNS

### Start the Service
```bash
sudo systemctl start named
systemctl status named
```

If it fails to start, check for typos in your config files (missing semicolons, periods, spaces).

---

### Configure Client to Use Your DNS Server

Edit `/etc/resolv.conf`:
```bash
sudo nano /etc/resolv.conf
```

Add your DNS server:
```
nameserver 192.168.8.2
```

Or add to netplan (`/etc/netplan/*.yaml`):
```yaml
nameservers:
  addresses:
    - 192.168.8.2
```

---

## Testing DNS

### Using nslookup

Forward lookup:
```bash
nslookup www.ncaecybergames.org
```

Reverse lookup:
```bash
nslookup 192.168.8.2
```

### Using Browser
Navigate to `http://www.ncaecybergames.org` - should load your web server.

---

## Troubleshooting

### Service Won't Start
- Check for typos in zone files (missing semicolons, periods)
- Check file permissions (`644`) and ownership (`root:bind`)
- Check config syntax: `named-checkconf`
- Check zone file syntax: `named-checkzone ncaecybergames.org /etc/bind/zones/forward.ncaecybergames.org`

### DNS Not Resolving
- Verify service is running: `systemctl status named`
- Check `/etc/resolv.conf` has your DNS server listed
- Test with `nslookup` to isolate DNS vs other issues

### Common Mistakes
- Forgetting to increment serial number after changes
- Missing periods at end of domain names in reverse zone files
- Wrong file permissions/ownership
- Typos in IP addresses or domain names
- Missing semicolons in config files

---

## Quick Reference

| Task | Command |
|------|---------|
| Check service status | `systemctl status named` |
| Start service | `sudo systemctl start named` |
| Restart after config change | `sudo systemctl restart named` |
| Check config syntax | `named-checkconf` |
| Check zone syntax | `named-checkzone DOMAIN ZONEFILE` |
| Forward lookup | `nslookup DOMAIN` |
| Reverse lookup | `nslookup IP` |
