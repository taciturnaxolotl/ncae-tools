# UFW Firewall Configuration

## Overview
UFW (Uncomplicated Firewall) sits on top of iptables and provides a more user-friendly interface for managing firewall rules on Ubuntu systems.

## Distribution Differences

| Distribution | Firewall Tool |
|--------------|---------------|
| Ubuntu | UFW (built-in) |
| Kali | iptables (UFW not installed by default) |
| CentOS/RHEL | firewall-cmd (firewalld) |

## Basic Commands

### Check Status
```bash
sudo ufw status              # Basic status
sudo ufw status verbose      # Detailed status with default policies
sudo ufw status numbered     # Show rule numbers
```

### Enable/Disable
```bash
sudo ufw enable              # Turn on firewall (persists after reboot)
sudo ufw disable             # Turn off firewall
```

## Default Policies
When you enable UFW, default behavior is:
- **Incoming**: DENY (block all incoming traffic by default)
- **Outgoing**: ALLOW (allow all outgoing traffic)
- **Routed**: DENY (no routing/forwarding)

This means services won't be accessible until you explicitly allow them.

## Creating Rules

### Allow Rules - By Service Name
```bash
sudo ufw allow ssh           # Allow SSH (port 22, IPv4 and IPv6)
sudo ufw allow http          # Allow HTTP (port 80)
sudo ufw allow https         # Allow HTTPS (port 443)
```

### Allow Rules - By Port
```bash
sudo ufw allow 22/tcp        # Allow TCP port 22
sudo ufw allow 80/tcp        # Allow TCP port 80
sudo ufw allow 53/udp        # Allow UDP port 53 (DNS)
```

### Allow Rules - By IP Address
```bash
sudo ufw allow from 192.168.1.100               # Allow all traffic from specific IP
sudo ufw allow from 192.168.1.0/24              # Allow from entire subnet
```

### Deny Rules
```bash
sudo ufw deny from 192.168.195.0/24             # Block entire subnet
sudo ufw deny 23/tcp                             # Block telnet
```

## Rule Processing Order

**Critical**: UFW processes rules in the order they were added.

```bash
# Example 1 - This works (allow processed first)
sudo ufw allow from 192.168.195.100
sudo ufw deny from 192.168.195.0/24
# Result: .100 is allowed, rest of subnet blocked

# Example 2 - This doesn't work as intended (deny processed first)
sudo ufw deny from 192.168.195.0/24
sudo ufw allow from 192.168.195.100
# Result: .100 is also blocked (caught by first deny rule)
```

## Deleting Rules

### By Rule Number
```bash
sudo ufw status numbered     # See rule numbers
sudo ufw delete 4            # Delete rule #4
```

**Warning**: After deleting a rule, all rules are renumbered. Delete one at a time and re-check numbers.

### By Specification
```bash
sudo ufw delete allow ssh
sudo ufw delete allow from 192.168.1.100
```

## IPv6 Considerations

Many UFW commands automatically create both IPv4 and IPv6 rules:

```bash
sudo ufw allow ssh
# Creates BOTH:
# - Port 22 (IPv4)
# - Port 22 (IPv6)
```

**Security Tip**: If you're not using IPv6, consider deleting those rules to reduce attack surface:
```bash
sudo ufw status numbered
sudo ufw delete 4            # Delete the IPv6 rule
```

## Before/After Rules

UFW has built-in rules that process **before** and **after** your user-defined rules. These are stored in:
- `/etc/ufw/before.rules` - Processed before user rules
- `/etc/ufw/after.rules` - Processed after user rules

Example before-rules:
- Allow DHCP client (so you can get an IP)
- Allow established connections
- Allow loopback traffic

You can edit these files if needed, but typically user rules are sufficient.

## Common Service Configurations

### SSH Server
```bash
sudo ufw allow ssh
# or
sudo ufw allow 22/tcp
```

### Web Server (Apache/Nginx)
```bash
sudo ufw allow http
sudo ufw allow https
# or
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### DNS Server
```bash
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
```

## Competition Tips

1. **Start by enabling it**: `sudo ufw enable` - even basic defaults improve security
2. **Allow services incrementally**: Only open ports for services you're actually running
3. **Check after each change**: `sudo ufw status verbose`
4. **Don't lock yourself out**: If configuring SSH remotely, make sure you allow SSH before enabling the firewall
5. **Monitor conflicts**: If a service stops working after enabling UFW, you likely forgot to allow its port

## Troubleshooting

### Service not accessible after enabling firewall
```bash
sudo ufw status numbered     # Check if port is allowed
sudo ufw allow <port>/tcp    # Add the missing rule
```

### Locked out of SSH
- If you have console access: `sudo ufw allow ssh` then `sudo ufw enable`
- Always add SSH rule before enabling firewall on remote systems

### Rule not working as expected
- Check rule order with `sudo ufw status numbered`
- More specific rules should come before general deny rules
- Remember: first match wins

## Integration with System Services

UFW rules persist across reboots once enabled. The firewall starts automatically on boot if you've run `sudo ufw enable`.

To disable automatic start:
```bash
sudo ufw disable
```
