# MikroTik Router Configuration

## Overview
Starting 2025, the NCAE competition replaced CentOS routers with MikroTik routers. MikroTik provides both a CLI and web GUI for configuration.

## Why MikroTik?
- CentOS is end-of-life
- MikroTik is a commercial router OS used in real networks
- Provides both CLI and web interface
- More intuitive than raw iptables

## Access Methods

### CLI Access (Console/Terminal)
- Through ProxMox VNC console
- Direct terminal access
- No browser required

### Web GUI Access
```
http://<router-ip>:8080
```

**Example**: `http://172.20.213.1:8080` (from external side)

**Port 8080** is the management interface, not the standard web port.

## Initial Login

### Default Credentials
- **Username**: `admin`
- **Password**: (blank - just press Enter)

### First Login
1. Login with blank password
2. System will prompt you to set a new password
3. **IMPORTANT**: Choose a strong password for competition
   - For testing/practice: can use something simple like `password`
   - For competition: red team will own you with weak passwords

### License Prompt
- Will ask if you want to view license
- Can say "no" unless interested

## Basic CLI Commands

### Check IP Addresses
```bash
/ip address print
```

Shows all configured IP addresses on all interfaces.

### Check Interfaces (Hardware)
```bash
interface print
```

Shows network adapters:
- `ether3` = First interface (usually external)
- `ether4` = Second interface (usually internal)
- Names may vary depending on hardware/cloning

### Assign an IP Address
```bash
/ip address add address=172.20.213.1/16 interface=ether3
```

**Breakdown**:
- `address=` - IP and subnet mask in CIDR notation
- `interface=` - Which network adapter (ether3, ether4, etc.)

**Example for internal side**:
```bash
/ip address add address=192.168.213.1/24 interface=ether4
```

### Test Connectivity
```bash
/ping 172.20.2
/ping 192.168.213.2
```

**Keyboard shortcuts**:
- Up/Down arrows = Command history
- Ctrl+C = Stop ping

### Check Configuration
Use the print command for any section:
```bash
/ip address print
/ip route print
/ip firewall nat print
```

## Web GUI Configuration

### Accessing the GUI

From external network:
```
http://172.20.213.1:8080
```

Login: `admin` / `<your-password>`

### GUI Navigation

**Top-right buttons**:
- **Quick Set** - Main configuration page (most common tasks)
- **Advanced** - Detailed/expert settings
- **Terminal** - CLI access from web browser

**Most tasks can be done from Quick Set.**

### Quick Set Configuration

**Scrolling tips**:
- Mouse wheel only works when cursor is in the CENTER of the page
- If scrolling doesn't work, move mouse to the left side
- Scroll bar appears in the middle column

#### Internet/External Configuration

**Gateway** (where traffic goes to reach internet):
```
172.20.1.1              # Or whatever your competition topology specifies
```

**DNS Servers**:
- Click the `+` button to add DNS servers
- Add all DNS servers from your topology document

#### LAN/Internal Configuration

Should show your configured internal IP:
```
192.168.213.1/24
```

#### Critical Checkboxes

✅ **Bridge LAN Ports** - Check this
- Allows multiple LAN ports to work as one network

✅ **Enable NAT** - Check this  
- **Network Address Translation**
- Allows internal 192.168.x.x addresses to route through external 172.20.x.x
- **Required for routing to work**

#### Apply Changes

Click **Apply Configuration** button at bottom.

Changes apply immediately - you'll see a "Saved" notification in the bottom-right.

### Port Forwarding (Port Mapping)

**Purpose**: Route external traffic to internal servers

**Example**: Route external HTTP requests to internal web server

1. Click **Port Mapping** (in Quick Set view)

2. Click **New** button

3. Configure the rule:

**TCP Rule**:
```
Name:        www-tcp
Protocol:    TCP
Port:        80
Forward To:  192.168.213.2
Port:        80
```

**UDP Rule**:
```
Name:        www-udp  
Protocol:    UDP
Port:        80
Forward To:  192.168.213.2
Port:        80
```

4. Click **OK** to save each rule

### Testing Port Forwarding

From external machine:
```
http://172.20.213.1
```

Should display website hosted on 192.168.213.2 (internal server).

## Mini-Hack Context

### External Network
```
Network: 172.20.0.0/16
Router IP: 172.20.213.1        (example team 213)
Kali External: 172.20.2
```

### Internal Network  
```
Network: 192.168.213.0/24      (team number in 3rd octet)
Router IP: 192.168.213.1
Web Server: 192.168.213.2
Kali Internal: 192.168.213.100
```

### Required Configuration

1. **Assign external IP**: `172.20.<team>.1/16` to ether3
2. **Assign internal IP**: `192.168.<team>.1/24` to ether4
3. **Enable NAT** in Quick Set
4. **Port forward 80** (TCP & UDP) to internal web server at `.2`

## Common Issues

### Can't access web GUI
- Verify router IP is correct
- Must use port 8080: `http://<ip>:8080`
- Check you're on the same network as router

### Port forwarding not working
- Did you enable NAT? (checkbox in Quick Set)
- Did you create BOTH TCP and UDP rules?
- Verify internal server is actually running the service
- Check internal server IP is correct

### Changes not saving
- Look for "Saved" notification bottom-right
- If using Quick Set, click "Apply Configuration"
- Changes are immediate (no reboot needed)

## CLI vs Web GUI

**Use CLI for**:
- Quick IP configuration
- Checking current status
- When GUI is not accessible

**Use Web GUI for**:
- Port forwarding / NAT rules
- Complex firewall rules
- Overview of configuration
- When you want visual confirmation

Both methods work and changes sync between them.

## Advanced Topics (Beyond Basics)

**Firewall Rules** - More complex than just port forwarding
- Can create allow/deny rules
- Similar concept to UFW but different syntax

**DHCP Server** - Assign IPs to internal network automatically
- Not needed for mini-hack (static IPs used)

**Routing Tables** - Custom routes
- Can add static routes for complex topologies

**VLANs** - Virtual network segmentation
- Competition may use in advanced scenarios

These are covered in MikroTik documentation but not required for basic mini-hack completion.

## Competition Day Checklist

1. ✅ Login and set a **strong** password
2. ✅ Assign external IP address to ether3
3. ✅ Assign internal IP address to ether4  
4. ✅ Configure gateway (from topology doc)
5. ✅ Add DNS servers (from topology doc)
6. ✅ Enable NAT checkbox
7. ✅ Create port forwarding rules for required services
8. ✅ Test connectivity from external network

## Resources

**Official Documentation**:
- [MikroTik Wiki](https://wiki.mikrotik.com/)
- [Getting Started Guide](https://wiki.mikrotik.com/wiki/Manual:First_time_startup)

**Search Tips**:
- "mikrotik quick set"
- "mikrotik port forwarding"
- "mikrotik NAT configuration"

Most common tasks are well-documented with examples.
