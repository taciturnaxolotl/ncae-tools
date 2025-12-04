# Network Configuration by Distribution

## Viewing Current Configuration

Show IP addresses:
```bash
ip a
# or
ip addr show
```

Show specific interface:
```bash
ip a show eth0
```

## Kali/Debian - /etc/network/interfaces

**File**: `/etc/network/interfaces`

Basic static configuration:
```bash
auto eth0
iface eth0 inet static
    address 172.20.118.100
    netmask 255.255.255.0
    gateway 172.20.118.1
```

**Key components:**
- `auto eth0` - Bring up interface automatically on boot
- `iface eth0 inet static` - Configure static IP (not DHCP)
- `address` - IP address
- `netmask` - Subnet mask
- `gateway` - Default gateway (router)

**Restart networking:**
```bash
sudo systemctl restart networking
# or
sudo ifdown eth0 && sudo ifup eth0
```

## CentOS/RHEL - ifcfg Files

**Directory**: `/etc/sysconfig/network-scripts/`

**Files**: One per interface (e.g., `ifcfg-eth0`, `ifcfg-eth1`)

Example `ifcfg-eth0`:
```bash
DEVICE=eth0
BOOTPROTO=static
ONBOOT=yes
IPADDR=172.20.118.1
NETMASK=255.255.255.0
GATEWAY=172.20.118.254
```

**Key settings:**
- `DEVICE` - Interface name
- `BOOTPROTO` - `static` or `dhcp`
- `ONBOOT` - `yes` to auto-start on boot
- `IPADDR` - IP address
- `NETMASK` - Subnet mask
- `GATEWAY` - Default gateway

**Restart networking:**
```bash
sudo systemctl restart network
# or per-interface:
sudo ifdown eth0 && sudo ifup eth0
```

## Ubuntu - Netplan (YAML)

**Directory**: `/etc/netplan/`

**File**: Usually `01-network-manager-all.yaml` (or similar `.yaml` file)

**IMPORTANT**: YAML is whitespace-sensitive. Use 2-space indentation consistently.

Example configuration:
```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens18:
      addresses:
        - 192.168.195.2/24
      gateway4: 192.168.195.1
```

**Key elements:**
- `ethernets:` - Section for ethernet interfaces
- `ens18:` - Interface name (not eth0 on modern Ubuntu)
- `addresses:` - List of IPs (note the dash and `/24` CIDR notation)
- `gateway4:` - Default gateway for IPv4

**Apply changes:**
```bash
sudo netplan apply
```

**Test configuration (doesn't persist):**
```bash
sudo netplan try
```

**CIDR notation:** `/24` equals `255.255.255.0`

## Temporary IP Configuration

Set IP temporarily (lost on reboot):
```bash
sudo ip addr add 192.168.1.100/24 dev eth0
```

Flush (remove) all IPs from interface:
```bash
sudo ip addr flush dev eth0
```

## Common Network Issues

1. **Wrong interface name**: Check with `ip a` first
2. **Typo in config file**: Double-check spelling and syntax
3. **Forgotten gateway**: Can't reach beyond local network
4. **Netplan spacing**: YAML requires exact indentation
5. **Wrong subnet**: Devices must be on same subnet to communicate

## Interface Naming

- **Old style**: `eth0`, `eth1`, `lo` (loopback)
- **New style**: `ens18`, `enp0s3`, etc. (Ubuntu/modern systems)
- Always check actual names with `ip a` before configuring
