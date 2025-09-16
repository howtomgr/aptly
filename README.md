# aptly Installation Guide

aptly is a free and open-source Debian repository manager. Aptly provides Debian repository management with snapshots and mirrors

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
  - CPU: 1 core minimum
  - RAM: 1GB minimum
  - Storage: 50GB for packages
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default aptly port)
  - None
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

# Install aptly
sudo dnf install -y aptly

# Enable and start service
sudo systemctl enable --now aptly

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
aptly --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install aptly
sudo apt install -y aptly

# Enable and start service
sudo systemctl enable --now aptly

# Configure firewall
sudo ufw allow 8080

# Verify installation
aptly --version
```

### Arch Linux

```bash
# Install aptly
sudo pacman -S aptly

# Enable and start service
sudo systemctl enable --now aptly

# Verify installation
aptly --version
```

### Alpine Linux

```bash
# Install aptly
apk add --no-cache aptly

# Enable and start service
rc-update add aptly default
rc-service aptly start

# Verify installation
aptly --version
```

### openSUSE/SLES

```bash
# Install aptly
sudo zypper install -y aptly

# Enable and start service
sudo systemctl enable --now aptly

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
aptly --version
```

### macOS

```bash
# Using Homebrew
brew install aptly

# Start service
brew services start aptly

# Verify installation
aptly --version
```

### FreeBSD

```bash
# Using pkg
pkg install aptly

# Enable in rc.conf
echo 'aptly_enable="YES"' >> /etc/rc.conf

# Start service
service aptly start

# Verify installation
aptly --version
```

### Windows

```bash
# Using Chocolatey
choco install aptly

# Or using Scoop
scoop install aptly

# Verify installation
aptly --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/aptly

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
aptly --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable aptly

# Start service
sudo systemctl start aptly

# Stop service
sudo systemctl stop aptly

# Restart service
sudo systemctl restart aptly

# Check status
sudo systemctl status aptly

# View logs
sudo journalctl -u aptly -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add aptly default

# Start service
rc-service aptly start

# Stop service
rc-service aptly stop

# Restart service
rc-service aptly restart

# Check status
rc-service aptly status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'aptly_enable="YES"' >> /etc/rc.conf

# Start service
service aptly start

# Stop service
service aptly stop

# Restart service
service aptly restart

# Check status
service aptly status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start aptly
brew services stop aptly
brew services restart aptly

# Check status
brew services list | grep aptly
```

### Windows Service Manager

```powershell
# Start service
net start aptly

# Stop service
net stop aptly

# Using PowerShell
Start-Service aptly
Stop-Service aptly
Restart-Service aptly

# Check status
Get-Service aptly
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream aptly_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name aptly.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name aptly.example.com;

    ssl_certificate /etc/ssl/certs/aptly.example.com.crt;
    ssl_certificate_key /etc/ssl/private/aptly.example.com.key;

    location / {
        proxy_pass http://aptly_backend;
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
    ServerName aptly.example.com
    Redirect permanent / https://aptly.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName aptly.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/aptly.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/aptly.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend aptly_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/aptly.pem
    redirect scheme https if !{ ssl_fc }
    default_backend aptly_backend

backend aptly_backend
    balance roundrobin
    server aptly1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R aptly:aptly /etc/aptly
sudo chmod 750 /etc/aptly

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
sudo systemctl status aptly

# View logs
sudo journalctl -u aptly -f

# Monitor resource usage
top -p $(pgrep aptly)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/aptly"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/aptly-backup-$DATE.tar.gz" /etc/aptly /var/lib/aptly

echo "Backup completed: $BACKUP_DIR/aptly-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop aptly

# Restore from backup
tar -xzf /backup/aptly/aptly-backup-*.tar.gz -C /

# Start service
sudo systemctl start aptly
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u aptly -n 100
sudo tail -f /var/log/aptly/aptly.log

# Check configuration
aptly --version

# Check permissions
ls -la /etc/aptly
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
top -p $(pgrep aptly)

# Check disk I/O
iotop -p $(pgrep aptly)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  aptly:
    image: aptly:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/aptly
      - ./data:/var/lib/aptly
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update aptly

# Debian/Ubuntu
sudo apt update && sudo apt upgrade aptly

# Arch Linux
sudo pacman -Syu aptly

# Alpine Linux
apk update && apk upgrade aptly

# openSUSE
sudo zypper update aptly

# FreeBSD
pkg update && pkg upgrade aptly

# Always backup before updates
tar -czf /backup/aptly-pre-update-$(date +%Y%m%d).tar.gz /etc/aptly

# Restart after updates
sudo systemctl restart aptly
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/aptly

# Clean old logs
find /var/log/aptly -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/aptly
```

## Additional Resources

- Official Documentation: https://docs.aptly.org/
- GitHub Repository: https://github.com/aptly/aptly
- Community Forum: https://forum.aptly.org/
- Best Practices Guide: https://docs.aptly.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
