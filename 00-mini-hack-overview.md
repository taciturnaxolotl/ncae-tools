# Mini-Hack Quick Start Guide

## Network Topology

```
External Network (172.20.0.0/16)
├── Kali External: 172.20.0.2
├── Router External: 172.20.<team>.1
└── Scoring Server: 172.20.0.1

Internal Network (192.168.<team>.0/24)
├── Router Internal: 192.168.<team>.1
├── Ubuntu Web Server: 192.168.<team>.2
└── Kali Internal: 192.168.<team>.100
```

**Your team number** is randomly assigned on each deployment (e.g., 213, 195, etc.)

## Objectives (Turn Lights Green)

1. ✅ Router online - responds to ping on external IP
2. ✅ Web server accessible - HTTP traffic routes through router to internal server
3. ✅ Service running - Apache returns content from internal web server

## Step-by-Step Checklist

### 1. Find Your Team Number

**On Kali External**:

Check scoreboard at http://172.20.1 for team number (username: `sandbox` pass: `password`).

### 2. Configure Router

**Login to MikroTik** (via ProxMox console or SSH):

```bash
# Default login
admin
<press Enter for blank password>

# Set a password when prompted
<choose password>
```

**Assign IP addresses**:

```bash
/interface print

# External interface
/ip address add address=172.20.<team>.1/16 interface=ether#

# Internal interface
/ip address add address=192.168.<team>.1/24 interface=ether#

# Verify
/ip address print
```

The webgui can't be used here because there are no ip addresses assigned to it yet.

### 3. Configure Ubuntu Web Server

**Assign static IP**:

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
```

```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 192.168.<team>.2/24
      routes:
        - to: default
          via: 192.168.<team>.1
```

```bash
sudo netplan apply
ip addr show                    # Verify IP
ping 192.168.<team>.1          # Test router connectivity
```

or open settings -> enable wired and config there

**Start Apache**:

```bash
sudo systemctl restart apache2
sudo systemctl status apache2   # Should show "active (running)"
```

```bash
sudo vi /var/www/html/index.html # change team number
```

### 4. Configure Port Forwarding (Router)

**Web GUI**:

```
http://172.20.<team>.1:8080
```

1. Go to **Quick Set** → **Port Mapping** (enable nat here and change gateway to `172.20.1.1`)
2. Click **New**
   - Name: `www-tcp`
   - Protocol: `TCP`
   - Port: `80`
   - Forward To: `192.168.<team>.2`
   - Port: `80`
3. Click **OK**
4. Repeat for UDP:
   - Name: `www-udp`
   - Protocol: `UDP`
   - Port: `80`
   - Forward To: `192.168.<team>.2`
   - Port: `80`

### 5. Test From External Network

**On Kali External**:

```bash
ping 172.20.<team>.1                    # Router should respond
curl http://172.20.<team>.1             # Should show web content from internal server
```

**Check scoreboard**: `http://172.20.1`

All lights should be green!

## Configuration Files Reference

**Router**: Web GUI at `http://172.20.<team>.1:8080` or CLI via console

**Ubuntu Web Server**:

- Network: `/etc/netplan/01-network-manager-all.yaml`
- Apache: `sudo systemctl restart apache2`
- Website content: `/var/www/html/`

**Kali Machines**: For testing only, no configuration needed
