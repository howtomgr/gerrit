# gerrit Installation Guide

gerrit is a free and open-source code review system. Gerrit provides web-based code review for Git projects

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
  - RAM: 2GB minimum
  - Storage: 5GB for repos
  - Network: HTTP/SSH access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default gerrit port)
  - SSH on port 29418
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

# Install gerrit
sudo dnf install -y gerrit

# Enable and start service
sudo systemctl enable --now gerrit

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
gerrit --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gerrit
sudo apt install -y gerrit

# Enable and start service
sudo systemctl enable --now gerrit

# Configure firewall
sudo ufw allow 8080

# Verify installation
gerrit --version
```

### Arch Linux

```bash
# Install gerrit
sudo pacman -S gerrit

# Enable and start service
sudo systemctl enable --now gerrit

# Verify installation
gerrit --version
```

### Alpine Linux

```bash
# Install gerrit
apk add --no-cache gerrit

# Enable and start service
rc-update add gerrit default
rc-service gerrit start

# Verify installation
gerrit --version
```

### openSUSE/SLES

```bash
# Install gerrit
sudo zypper install -y gerrit

# Enable and start service
sudo systemctl enable --now gerrit

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
gerrit --version
```

### macOS

```bash
# Using Homebrew
brew install gerrit

# Start service
brew services start gerrit

# Verify installation
gerrit --version
```

### FreeBSD

```bash
# Using pkg
pkg install gerrit

# Enable in rc.conf
echo 'gerrit_enable="YES"' >> /etc/rc.conf

# Start service
service gerrit start

# Verify installation
gerrit --version
```

### Windows

```bash
# Using Chocolatey
choco install gerrit

# Or using Scoop
scoop install gerrit

# Verify installation
gerrit --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gerrit

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gerrit --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gerrit

# Start service
sudo systemctl start gerrit

# Stop service
sudo systemctl stop gerrit

# Restart service
sudo systemctl restart gerrit

# Check status
sudo systemctl status gerrit

# View logs
sudo journalctl -u gerrit -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gerrit default

# Start service
rc-service gerrit start

# Stop service
rc-service gerrit stop

# Restart service
rc-service gerrit restart

# Check status
rc-service gerrit status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gerrit_enable="YES"' >> /etc/rc.conf

# Start service
service gerrit start

# Stop service
service gerrit stop

# Restart service
service gerrit restart

# Check status
service gerrit status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gerrit
brew services stop gerrit
brew services restart gerrit

# Check status
brew services list | grep gerrit
```

### Windows Service Manager

```powershell
# Start service
net start gerrit

# Stop service
net stop gerrit

# Using PowerShell
Start-Service gerrit
Stop-Service gerrit
Restart-Service gerrit

# Check status
Get-Service gerrit
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gerrit_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name gerrit.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gerrit.example.com;

    ssl_certificate /etc/ssl/certs/gerrit.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gerrit.example.com.key;

    location / {
        proxy_pass http://gerrit_backend;
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
    ServerName gerrit.example.com
    Redirect permanent / https://gerrit.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gerrit.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gerrit.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gerrit.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gerrit_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gerrit.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gerrit_backend

backend gerrit_backend
    balance roundrobin
    server gerrit1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gerrit:gerrit /etc/gerrit
sudo chmod 750 /etc/gerrit

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status gerrit

# View logs
sudo journalctl -u gerrit -f

# Monitor resource usage
top -p $(pgrep gerrit)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gerrit"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gerrit-backup-$DATE.tar.gz" /etc/gerrit /var/lib/gerrit

echo "Backup completed: $BACKUP_DIR/gerrit-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gerrit

# Restore from backup
tar -xzf /backup/gerrit/gerrit-backup-*.tar.gz -C /

# Start service
sudo systemctl start gerrit
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gerrit -n 100
sudo tail -f /var/log/gerrit/gerrit.log

# Check configuration
gerrit --version

# Check permissions
ls -la /etc/gerrit
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gerrit)

# Check disk I/O
iotop -p $(pgrep gerrit)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gerrit:
    image: gerrit:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/gerrit
      - ./data:/var/lib/gerrit
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gerrit

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gerrit

# Arch Linux
sudo pacman -Syu gerrit

# Alpine Linux
apk update && apk upgrade gerrit

# openSUSE
sudo zypper update gerrit

# FreeBSD
pkg update && pkg upgrade gerrit

# Always backup before updates
tar -czf /backup/gerrit-pre-update-$(date +%Y%m%d).tar.gz /etc/gerrit

# Restart after updates
sudo systemctl restart gerrit
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gerrit

# Clean old logs
find /var/log/gerrit -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gerrit
```

## Additional Resources

- Official Documentation: https://docs.gerrit.org/
- GitHub Repository: https://github.com/gerrit/gerrit
- Community Forum: https://forum.gerrit.org/
- Best Practices Guide: https://docs.gerrit.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
