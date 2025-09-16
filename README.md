# kimchi Installation Guide

kimchi is a free and open-source KVM web interface. Kimchi provides HTML5 based management for KVM guests

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
  - Storage: 10GB for images
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8001 (default kimchi port)
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

# Install kimchi
sudo dnf install -y kimchi

# Enable and start service
sudo systemctl enable --now kimchi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8001/tcp
sudo firewall-cmd --reload

# Verify installation
kimchi --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kimchi
sudo apt install -y kimchi

# Enable and start service
sudo systemctl enable --now kimchi

# Configure firewall
sudo ufw allow 8001

# Verify installation
kimchi --version
```

### Arch Linux

```bash
# Install kimchi
sudo pacman -S kimchi

# Enable and start service
sudo systemctl enable --now kimchi

# Verify installation
kimchi --version
```

### Alpine Linux

```bash
# Install kimchi
apk add --no-cache kimchi

# Enable and start service
rc-update add kimchi default
rc-service kimchi start

# Verify installation
kimchi --version
```

### openSUSE/SLES

```bash
# Install kimchi
sudo zypper install -y kimchi

# Enable and start service
sudo systemctl enable --now kimchi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8001/tcp
sudo firewall-cmd --reload

# Verify installation
kimchi --version
```

### macOS

```bash
# Using Homebrew
brew install kimchi

# Start service
brew services start kimchi

# Verify installation
kimchi --version
```

### FreeBSD

```bash
# Using pkg
pkg install kimchi

# Enable in rc.conf
echo 'kimchi_enable="YES"' >> /etc/rc.conf

# Start service
service kimchi start

# Verify installation
kimchi --version
```

### Windows

```bash
# Using Chocolatey
choco install kimchi

# Or using Scoop
scoop install kimchi

# Verify installation
kimchi --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/kimchi

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
kimchi --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kimchi

# Start service
sudo systemctl start kimchi

# Stop service
sudo systemctl stop kimchi

# Restart service
sudo systemctl restart kimchi

# Check status
sudo systemctl status kimchi

# View logs
sudo journalctl -u kimchi -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kimchi default

# Start service
rc-service kimchi start

# Stop service
rc-service kimchi stop

# Restart service
rc-service kimchi restart

# Check status
rc-service kimchi status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kimchi_enable="YES"' >> /etc/rc.conf

# Start service
service kimchi start

# Stop service
service kimchi stop

# Restart service
service kimchi restart

# Check status
service kimchi status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kimchi
brew services stop kimchi
brew services restart kimchi

# Check status
brew services list | grep kimchi
```

### Windows Service Manager

```powershell
# Start service
net start kimchi

# Stop service
net stop kimchi

# Using PowerShell
Start-Service kimchi
Stop-Service kimchi
Restart-Service kimchi

# Check status
Get-Service kimchi
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kimchi_backend {
    server 127.0.0.1:8001;
}

server {
    listen 80;
    server_name kimchi.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kimchi.example.com;

    ssl_certificate /etc/ssl/certs/kimchi.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kimchi.example.com.key;

    location / {
        proxy_pass http://kimchi_backend;
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
    ServerName kimchi.example.com
    Redirect permanent / https://kimchi.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kimchi.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kimchi.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kimchi.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8001/
    ProxyPassReverse / http://127.0.0.1:8001/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kimchi_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kimchi.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kimchi_backend

backend kimchi_backend
    balance roundrobin
    server kimchi1 127.0.0.1:8001 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kimchi:kimchi /etc/kimchi
sudo chmod 750 /etc/kimchi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8001/tcp
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
sudo systemctl status kimchi

# View logs
sudo journalctl -u kimchi -f

# Monitor resource usage
top -p $(pgrep kimchi)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/kimchi"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/kimchi-backup-$DATE.tar.gz" /etc/kimchi /var/lib/kimchi

echo "Backup completed: $BACKUP_DIR/kimchi-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kimchi

# Restore from backup
tar -xzf /backup/kimchi/kimchi-backup-*.tar.gz -C /

# Start service
sudo systemctl start kimchi
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kimchi -n 100
sudo tail -f /var/log/kimchi/kimchi.log

# Check configuration
kimchi --version

# Check permissions
ls -la /etc/kimchi
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8001

# Test connectivity
telnet localhost 8001

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kimchi)

# Check disk I/O
iotop -p $(pgrep kimchi)

# Check connections
ss -an | grep 8001
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  kimchi:
    image: kimchi:latest
    ports:
      - "8001:8001"
    volumes:
      - ./config:/etc/kimchi
      - ./data:/var/lib/kimchi
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kimchi

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kimchi

# Arch Linux
sudo pacman -Syu kimchi

# Alpine Linux
apk update && apk upgrade kimchi

# openSUSE
sudo zypper update kimchi

# FreeBSD
pkg update && pkg upgrade kimchi

# Always backup before updates
tar -czf /backup/kimchi-pre-update-$(date +%Y%m%d).tar.gz /etc/kimchi

# Restart after updates
sudo systemctl restart kimchi
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/kimchi

# Clean old logs
find /var/log/kimchi -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/kimchi
```

## Additional Resources

- Official Documentation: https://docs.kimchi.org/
- GitHub Repository: https://github.com/kimchi/kimchi
- Community Forum: https://forum.kimchi.org/
- Best Practices Guide: https://docs.kimchi.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
