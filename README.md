# tinkerbell Installation Guide

tinkerbell is a free and open-source bare metal provisioning. Tinkerbell provides framework for bare metal provisioning

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
  - Storage: 50GB for workflows
  - Network: HTTP/DHCP/TFTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 42113 (default tinkerbell port)
  - Various services
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

# Install tinkerbell
sudo dnf install -y tinkerbell

# Enable and start service
sudo systemctl enable --now tinkerbell

# Configure firewall
sudo firewall-cmd --permanent --add-port=42113/tcp
sudo firewall-cmd --reload

# Verify installation
tinkerbell --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install tinkerbell
sudo apt install -y tinkerbell

# Enable and start service
sudo systemctl enable --now tinkerbell

# Configure firewall
sudo ufw allow 42113

# Verify installation
tinkerbell --version
```

### Arch Linux

```bash
# Install tinkerbell
sudo pacman -S tinkerbell

# Enable and start service
sudo systemctl enable --now tinkerbell

# Verify installation
tinkerbell --version
```

### Alpine Linux

```bash
# Install tinkerbell
apk add --no-cache tinkerbell

# Enable and start service
rc-update add tinkerbell default
rc-service tinkerbell start

# Verify installation
tinkerbell --version
```

### openSUSE/SLES

```bash
# Install tinkerbell
sudo zypper install -y tinkerbell

# Enable and start service
sudo systemctl enable --now tinkerbell

# Configure firewall
sudo firewall-cmd --permanent --add-port=42113/tcp
sudo firewall-cmd --reload

# Verify installation
tinkerbell --version
```

### macOS

```bash
# Using Homebrew
brew install tinkerbell

# Start service
brew services start tinkerbell

# Verify installation
tinkerbell --version
```

### FreeBSD

```bash
# Using pkg
pkg install tinkerbell

# Enable in rc.conf
echo 'tinkerbell_enable="YES"' >> /etc/rc.conf

# Start service
service tinkerbell start

# Verify installation
tinkerbell --version
```

### Windows

```bash
# Using Chocolatey
choco install tinkerbell

# Or using Scoop
scoop install tinkerbell

# Verify installation
tinkerbell --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/tinkerbell

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
tinkerbell --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable tinkerbell

# Start service
sudo systemctl start tinkerbell

# Stop service
sudo systemctl stop tinkerbell

# Restart service
sudo systemctl restart tinkerbell

# Check status
sudo systemctl status tinkerbell

# View logs
sudo journalctl -u tinkerbell -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add tinkerbell default

# Start service
rc-service tinkerbell start

# Stop service
rc-service tinkerbell stop

# Restart service
rc-service tinkerbell restart

# Check status
rc-service tinkerbell status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'tinkerbell_enable="YES"' >> /etc/rc.conf

# Start service
service tinkerbell start

# Stop service
service tinkerbell stop

# Restart service
service tinkerbell restart

# Check status
service tinkerbell status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start tinkerbell
brew services stop tinkerbell
brew services restart tinkerbell

# Check status
brew services list | grep tinkerbell
```

### Windows Service Manager

```powershell
# Start service
net start tinkerbell

# Stop service
net stop tinkerbell

# Using PowerShell
Start-Service tinkerbell
Stop-Service tinkerbell
Restart-Service tinkerbell

# Check status
Get-Service tinkerbell
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream tinkerbell_backend {
    server 127.0.0.1:42113;
}

server {
    listen 80;
    server_name tinkerbell.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name tinkerbell.example.com;

    ssl_certificate /etc/ssl/certs/tinkerbell.example.com.crt;
    ssl_certificate_key /etc/ssl/private/tinkerbell.example.com.key;

    location / {
        proxy_pass http://tinkerbell_backend;
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
    ServerName tinkerbell.example.com
    Redirect permanent / https://tinkerbell.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName tinkerbell.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/tinkerbell.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/tinkerbell.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:42113/
    ProxyPassReverse / http://127.0.0.1:42113/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend tinkerbell_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/tinkerbell.pem
    redirect scheme https if !{ ssl_fc }
    default_backend tinkerbell_backend

backend tinkerbell_backend
    balance roundrobin
    server tinkerbell1 127.0.0.1:42113 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R tinkerbell:tinkerbell /etc/tinkerbell
sudo chmod 750 /etc/tinkerbell

# Configure firewall
sudo firewall-cmd --permanent --add-port=42113/tcp
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
sudo systemctl status tinkerbell

# View logs
sudo journalctl -u tinkerbell -f

# Monitor resource usage
top -p $(pgrep tinkerbell)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/tinkerbell"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/tinkerbell-backup-$DATE.tar.gz" /etc/tinkerbell /var/lib/tinkerbell

echo "Backup completed: $BACKUP_DIR/tinkerbell-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop tinkerbell

# Restore from backup
tar -xzf /backup/tinkerbell/tinkerbell-backup-*.tar.gz -C /

# Start service
sudo systemctl start tinkerbell
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u tinkerbell -n 100
sudo tail -f /var/log/tinkerbell/tinkerbell.log

# Check configuration
tinkerbell --version

# Check permissions
ls -la /etc/tinkerbell
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 42113

# Test connectivity
telnet localhost 42113

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep tinkerbell)

# Check disk I/O
iotop -p $(pgrep tinkerbell)

# Check connections
ss -an | grep 42113
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  tinkerbell:
    image: tinkerbell:latest
    ports:
      - "42113:42113"
    volumes:
      - ./config:/etc/tinkerbell
      - ./data:/var/lib/tinkerbell
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update tinkerbell

# Debian/Ubuntu
sudo apt update && sudo apt upgrade tinkerbell

# Arch Linux
sudo pacman -Syu tinkerbell

# Alpine Linux
apk update && apk upgrade tinkerbell

# openSUSE
sudo zypper update tinkerbell

# FreeBSD
pkg update && pkg upgrade tinkerbell

# Always backup before updates
tar -czf /backup/tinkerbell-pre-update-$(date +%Y%m%d).tar.gz /etc/tinkerbell

# Restart after updates
sudo systemctl restart tinkerbell
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/tinkerbell

# Clean old logs
find /var/log/tinkerbell -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/tinkerbell
```

## Additional Resources

- Official Documentation: https://docs.tinkerbell.org/
- GitHub Repository: https://github.com/tinkerbell/tinkerbell
- Community Forum: https://forum.tinkerbell.org/
- Best Practices Guide: https://docs.tinkerbell.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
