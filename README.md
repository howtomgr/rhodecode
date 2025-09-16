# rhodecode Installation Guide

rhodecode is a free and open-source enterprise source code management. RhodeCode provides enterprise source code management platform

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 10GB for repos
  - Network: HTTP/SSH access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 10000 (default rhodecode port)
  - Various service ports
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install rhodecode
sudo dnf install -y rhodecode

# Enable and start service
sudo systemctl enable --now rhodecode

# Configure firewall
sudo firewall-cmd --permanent --add-port=10000/tcp
sudo firewall-cmd --reload

# Verify installation
rhodecode --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install rhodecode
sudo apt install -y rhodecode

# Enable and start service
sudo systemctl enable --now rhodecode

# Configure firewall
sudo ufw allow 10000

# Verify installation
rhodecode --version
```

### Arch Linux

```bash
# Install rhodecode
sudo pacman -S rhodecode

# Enable and start service
sudo systemctl enable --now rhodecode

# Verify installation
rhodecode --version
```

### Alpine Linux

```bash
# Install rhodecode
apk add --no-cache rhodecode

# Enable and start service
rc-update add rhodecode default
rc-service rhodecode start

# Verify installation
rhodecode --version
```

### openSUSE/SLES

```bash
# Install rhodecode
sudo zypper install -y rhodecode

# Enable and start service
sudo systemctl enable --now rhodecode

# Configure firewall
sudo firewall-cmd --permanent --add-port=10000/tcp
sudo firewall-cmd --reload

# Verify installation
rhodecode --version
```

### macOS

```bash
# Using Homebrew
brew install rhodecode

# Start service
brew services start rhodecode

# Verify installation
rhodecode --version
```

### FreeBSD

```bash
# Using pkg
pkg install rhodecode

# Enable in rc.conf
echo 'rhodecode_enable="YES"' >> /etc/rc.conf

# Start service
service rhodecode start

# Verify installation
rhodecode --version
```

### Windows

```bash
# Using Chocolatey
choco install rhodecode

# Or using Scoop
scoop install rhodecode

# Verify installation
rhodecode --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/rhodecode

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
rhodecode --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable rhodecode

# Start service
sudo systemctl start rhodecode

# Stop service
sudo systemctl stop rhodecode

# Restart service
sudo systemctl restart rhodecode

# Check status
sudo systemctl status rhodecode

# View logs
sudo journalctl -u rhodecode -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add rhodecode default

# Start service
rc-service rhodecode start

# Stop service
rc-service rhodecode stop

# Restart service
rc-service rhodecode restart

# Check status
rc-service rhodecode status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'rhodecode_enable="YES"' >> /etc/rc.conf

# Start service
service rhodecode start

# Stop service
service rhodecode stop

# Restart service
service rhodecode restart

# Check status
service rhodecode status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start rhodecode
brew services stop rhodecode
brew services restart rhodecode

# Check status
brew services list | grep rhodecode
```

### Windows Service Manager

```powershell
# Start service
net start rhodecode

# Stop service
net stop rhodecode

# Using PowerShell
Start-Service rhodecode
Stop-Service rhodecode
Restart-Service rhodecode

# Check status
Get-Service rhodecode
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream rhodecode_backend {
    server 127.0.0.1:10000;
}

server {
    listen 80;
    server_name rhodecode.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rhodecode.example.com;

    ssl_certificate /etc/ssl/certs/rhodecode.example.com.crt;
    ssl_certificate_key /etc/ssl/private/rhodecode.example.com.key;

    location / {
        proxy_pass http://rhodecode_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName rhodecode.example.com
    Redirect permanent / https://rhodecode.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName rhodecode.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rhodecode.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/rhodecode.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:10000/
    ProxyPassReverse / http://127.0.0.1:10000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend rhodecode_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/rhodecode.pem
    redirect scheme https if !{ ssl_fc }
    default_backend rhodecode_backend

backend rhodecode_backend
    balance roundrobin
    server rhodecode1 127.0.0.1:10000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R rhodecode:rhodecode /etc/rhodecode
sudo chmod 750 /etc/rhodecode

# Configure firewall
sudo firewall-cmd --permanent --add-port=10000/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status rhodecode

# View logs
sudo journalctl -u rhodecode -f

# Monitor resource usage
top -p $(pgrep rhodecode)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/rhodecode"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/rhodecode-backup-$DATE.tar.gz" /etc/rhodecode /var/lib/rhodecode

echo "Backup completed: $BACKUP_DIR/rhodecode-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop rhodecode

# Restore from backup
tar -xzf /backup/rhodecode/rhodecode-backup-*.tar.gz -C /

# Start service
sudo systemctl start rhodecode
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u rhodecode -n 100
sudo tail -f /var/log/rhodecode/rhodecode.log

# Check configuration
rhodecode --version

# Check permissions
ls -la /etc/rhodecode
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 10000

# Test connectivity
telnet localhost 10000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep rhodecode)

# Check disk I/O
iotop -p $(pgrep rhodecode)

# Check connections
ss -an | grep 10000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  rhodecode:
    image: rhodecode:latest
    ports:
      - "10000:10000"
    volumes:
      - ./config:/etc/rhodecode
      - ./data:/var/lib/rhodecode
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update rhodecode

# Debian/Ubuntu
sudo apt update && sudo apt upgrade rhodecode

# Arch Linux
sudo pacman -Syu rhodecode

# Alpine Linux
apk update && apk upgrade rhodecode

# openSUSE
sudo zypper update rhodecode

# FreeBSD
pkg update && pkg upgrade rhodecode

# Always backup before updates
tar -czf /backup/rhodecode-pre-update-$(date +%Y%m%d).tar.gz /etc/rhodecode

# Restart after updates
sudo systemctl restart rhodecode
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/rhodecode

# Clean old logs
find /var/log/rhodecode -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/rhodecode
```

## Additional Resources

- Official Documentation: https://docs.rhodecode.org/
- GitHub Repository: https://github.com/rhodecode/rhodecode
- Community Forum: https://forum.rhodecode.org/
- Best Practices Guide: https://docs.rhodecode.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
