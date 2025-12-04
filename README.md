# Linux Service Configuration Writeups

Quick reference guides for configuring services in Linux competitions. Assumes basic Linux knowledge (filesystem navigation, systemctl, ssh, etc.).

## Writeups

0. **[Mini-Hack Quick Start](00-mini-hack-overview.md)** - Complete mini-hack walkthrough checklist
1. **[Services Overview](01-services-overview.md)** - General approach to any service
2. **[Apache Web Service](02-apache-web-service.md)** - HTTP/HTTPS server configuration
3. **[SSH Service](03-ssh-service.md)** - Remote access, keys, security
4. **[Network Configuration](04-network-configuration.md)** - Static IPs across different distros
5. **[DNS, Rsync, Cron](05-dns-rsync-cron.md)** - Name resolution and automated backups
6. **[UFW Firewall](06-ufw-firewall.md)** - Ubuntu firewall configuration
7. **[Active Connection Defense](07-active-connection-defense.md)** - Monitor and kill malicious connections
8. **[MikroTik Router](08-mikrotik-router.md)** - Router configuration (2025 competition)

## Service-Specific Quick Reference

### Apache Service Names
```bash
apache2      # Ubuntu/Debian/Kali
httpd        # CentOS/RHEL
```

### Network Configuration Files

| Distribution | Config Location |
|--------------|----------------|
| Kali/Debian  | `/etc/network/interfaces` |
| Ubuntu       | `/etc/netplan/*.yaml` |
| CentOS/RHEL  | `/etc/sysconfig/network-scripts/ifcfg-*` |

### SSH Key Permissions
```bash
chmod 700 ~/.ssh/
chmod 600 ~/.ssh/id_rsa          # Private key
chmod 644 ~/.ssh/id_rsa.pub      # Public key
chmod 644 ~/.ssh/authorized_keys
```

Regenerate host keys on cloned VMs:
```bash
sudo ssh-keygen -A
sudo systemctl restart sshd
```

### UFW Firewall
```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow from 192.168.1.100      # Specific IP
sudo ufw deny from 192.168.1.0/24      # Entire subnet
sudo ufw status numbered                # See rule numbers
sudo ufw delete 4                       # Delete rule by number
```

### Active Connection Monitoring
```bash
sudo netstat -tunap                     # All connections with PIDs
sudo netstat -tunap | grep ESTABLISHED  # Only active
w                                       # Who is logged in
sudo kill <PID>                         # Kill by process ID
sudo pkill -kill -u username            # Kill all user processes
```

### MikroTik Router
**CLI**:
```bash
/ip address print
/ip address add address=192.168.1.1/24 interface=ether3
/ping 192.168.1.2
interface print
```

**Web GUI**: `http://<router-ip>:8080`  
Default login: `admin` / (blank password)

### Rsync + Cron
**Rsync common patterns**:
```bash
rsync -av source/ dest/                             # Basic sync
rsync -av --delete source/ dest/                    # Mirror (delete extra files in dest)
rsync -avz local/ user@host:remote/                 # Remote backup (z=compress)
rsync -av --exclude='*.log' source/ dest/           # Exclude files
rsync -av source/ dest/ --dry-run                   # Test without changes
```

**Cron syntax**: `minute hour day month weekday command`
```
0 2 * * * /path/to/backup.sh      # Daily at 2 AM
*/15 * * * * /path/to/script.sh   # Every 15 minutes
0 */6 * * * rsync -av /data/ /backup/  # Every 6 hours
```

## Distribution Differences

| Feature | Ubuntu | Kali | CentOS/RHEL |
|---------|--------|------|-------------|
| Apache service | `apache2` | `apache2` | `httpd` |
| Network config | netplan YAML | interfaces | ifcfg-* scripts |
| Firewall | UFW | iptables | firewall-cmd |
| Cron service | `cron` | `cron` | `crond` |

**Router (2025)**: All distributions use MikroTik (replaces CentOS router)

## Competition Tips

1. **Network config varies by distro** - check which one first
2. **SSH keys**: Regenerate on cloned VMs, fix permissions (700/.ssh, 600/private)
3. **Enable firewall early** - UFW even with defaults improves security
4. **Monitor active connections** - assign someone to watch `netstat -tunap`
5. **Router (2025)**: MikroTik web GUI on port 8080, must enable NAT checkbox
6. **Port forwarding**: Create both TCP and UDP rules for most services
7. **Kill by PID not username** if you share accounts with red team
8. **Backup configs before changes** - especially network configs (can lock yourself out)

## Critical Configuration Locations

| Service | Config File(s) |
|---------|---------------|
| SSH | `/etc/ssh/sshd_config` |
| Apache (Ubuntu) | `/etc/apache2/apache2.conf`, `/etc/apache2/sites-available/` |
| Apache (CentOS) | `/etc/httpd/conf/httpd.conf`, `/etc/httpd/conf.d/` |
| Network (Kali) | `/etc/network/interfaces` |
| Network (Ubuntu) | `/etc/netplan/*.yaml` |
| Network (CentOS) | `/etc/sysconfig/network-scripts/ifcfg-*` |
| DNS resolution | `/etc/resolv.conf` |
| Cron jobs | `crontab -e` (per-user), `/etc/crontab` (system-wide) |
