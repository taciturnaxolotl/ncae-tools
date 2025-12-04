# Apache Web Service

## Service Name
- `apache2` (Ubuntu/Debian)
- `httpd` (CentOS/RHEL)

## Check Service Status
```bash
systemctl status apache2    # Ubuntu
systemctl status httpd      # CentOS
```

## Configuration Locations

Main config: `/etc/apache2/` (Ubuntu) or `/etc/httpd/` (CentOS)

Key files:
- `/etc/apache2/apache2.conf` - Main configuration
- `/etc/apache2/sites-available/` - Available site configs
- `/etc/apache2/sites-enabled/` - Active site configs (usually symlinks)

## Default Site Configuration

File: `/etc/apache2/sites-available/000-default.conf`

Key directives:
```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html
    # ... other settings
</VirtualHost>
```

- **Listen port**: Default is `*:80` (any IP, port 80)
- **DocumentRoot**: `/var/www/html` - where website files live

## Website File Location

Website files go in: `/var/www/html`

Default file: `index.html` (or `index.php`)

The web server automatically serves `index.html` when you visit the root URL.

## Start/Restart Service

```bash
sudo systemctl start apache2
sudo systemctl restart apache2
```

## Creating Website Content

Make directories:
```bash
sudo mkdir /var/www/html/newfolder
```

Create files:
```bash
sudo touch /var/www/html/newfile.html
```

**Permission Requirements**: Web server needs read permissions to serve files.

## Security Considerations

- Don't put sensitive files (like `/etc/shadow`) in `/var/www/html`
- Check permissions - files need to be readable by web server
- Backup config files before making changes
- The website displays actual files from the server's filesystem

## Common Issues

1. **Service not starting**: Check config file syntax
2. **Can't access website**: Verify service is running, check IP/port
3. **404 errors**: Check DocumentRoot path and file permissions
4. **Permission denied**: Files need world-readable permissions for web server access
