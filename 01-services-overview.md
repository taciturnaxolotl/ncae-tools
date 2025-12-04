# Linux Services - General Approach

## Service Configuration Checklist

When encountering any new service:

1. **Understand what it does** - Don't rush into clicking buttons. Read documentation first. Even 5 minutes of research saves time later.

2. **Locate configuration files** - Services usually have config files in `/etc`. Files can be singular or multiple across different locations (main config + user-specific).

3. **Backup before changes** - Always copy config files before modifying:
   ```bash
   sudo cp /etc/service/config /etc/service/config.bak
   ```

4. **Restart after changes** - Most services require restart for changes to take effect:
   ```bash
   sudo systemctl restart <service-name>
   ```
   Don't restart the entire computer - restart just the service.

5. **Check service status** - Verify if service is running:
   ```bash
   systemctl status <service-name>
   ```

6. **Dependencies matter** - Some services rely on others. Changing one may require restarting dependent services.

## Service Management Commands

Check service status (no sudo needed):
```bash
systemctl status <service-name>
```

Start a service:
```bash
sudo systemctl start <service-name>
```

Stop a service:
```bash
sudo systemctl stop <service-name>
```

Restart a service:
```bash
sudo systemctl restart <service-name>
```

Enable service to start on boot:
```bash
sudo systemctl enable <service-name>
```

Check if service is enabled:
```bash
systemctl is-enabled <service-name>
```
