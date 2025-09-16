# openvpn Installation Guide

openvpn is a free and open-source full-featured SSL VPN solution. OpenVPN provides secure point-to-point or site-to-site connections, serving as an open-source alternative to commercial VPN solutions

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
  - RAM: 256MB minimum
  - Storage: 100MB for installation
  - Network: UDP/TCP connectivity
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1194 (default openvpn port)
  - TCP 443 as fallback
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

# Install openvpn
sudo dnf install -y openvpn

# Enable and start service
sudo systemctl enable --now openvpn

# Configure firewall
sudo firewall-cmd --permanent --add-port=1194/tcp
sudo firewall-cmd --reload

# Verify installation
openvpn --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install openvpn
sudo apt install -y openvpn

# Enable and start service
sudo systemctl enable --now openvpn

# Configure firewall
sudo ufw allow 1194

# Verify installation
openvpn --version
```

### Arch Linux

```bash
# Install openvpn
sudo pacman -S openvpn

# Enable and start service
sudo systemctl enable --now openvpn

# Verify installation
openvpn --version
```

### Alpine Linux

```bash
# Install openvpn
apk add --no-cache openvpn

# Enable and start service
rc-update add openvpn default
rc-service openvpn start

# Verify installation
openvpn --version
```

### openSUSE/SLES

```bash
# Install openvpn
sudo zypper install -y openvpn

# Enable and start service
sudo systemctl enable --now openvpn

# Configure firewall
sudo firewall-cmd --permanent --add-port=1194/tcp
sudo firewall-cmd --reload

# Verify installation
openvpn --version
```

### macOS

```bash
# Using Homebrew
brew install openvpn

# Start service
brew services start openvpn

# Verify installation
openvpn --version
```

### FreeBSD

```bash
# Using pkg
pkg install openvpn

# Enable in rc.conf
echo 'openvpn_enable="YES"' >> /etc/rc.conf

# Start service
service openvpn start

# Verify installation
openvpn --version
```

### Windows

```bash
# Using Chocolatey
choco install openvpn

# Or using Scoop
scoop install openvpn

# Verify installation
openvpn --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/openvpn

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
openvpn --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable openvpn

# Start service
sudo systemctl start openvpn

# Stop service
sudo systemctl stop openvpn

# Restart service
sudo systemctl restart openvpn

# Check status
sudo systemctl status openvpn

# View logs
sudo journalctl -u openvpn -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add openvpn default

# Start service
rc-service openvpn start

# Stop service
rc-service openvpn stop

# Restart service
rc-service openvpn restart

# Check status
rc-service openvpn status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'openvpn_enable="YES"' >> /etc/rc.conf

# Start service
service openvpn start

# Stop service
service openvpn stop

# Restart service
service openvpn restart

# Check status
service openvpn status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start openvpn
brew services stop openvpn
brew services restart openvpn

# Check status
brew services list | grep openvpn
```

### Windows Service Manager

```powershell
# Start service
net start openvpn

# Stop service
net stop openvpn

# Using PowerShell
Start-Service openvpn
Stop-Service openvpn
Restart-Service openvpn

# Check status
Get-Service openvpn
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream openvpn_backend {
    server 127.0.0.1:1194;
}

server {
    listen 80;
    server_name openvpn.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name openvpn.example.com;

    ssl_certificate /etc/ssl/certs/openvpn.example.com.crt;
    ssl_certificate_key /etc/ssl/private/openvpn.example.com.key;

    location / {
        proxy_pass http://openvpn_backend;
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
    ServerName openvpn.example.com
    Redirect permanent / https://openvpn.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName openvpn.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/openvpn.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/openvpn.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1194/
    ProxyPassReverse / http://127.0.0.1:1194/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend openvpn_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/openvpn.pem
    redirect scheme https if !{ ssl_fc }
    default_backend openvpn_backend

backend openvpn_backend
    balance roundrobin
    server openvpn1 127.0.0.1:1194 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R openvpn:openvpn /etc/openvpn
sudo chmod 750 /etc/openvpn

# Configure firewall
sudo firewall-cmd --permanent --add-port=1194/tcp
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
sudo systemctl status openvpn

# View logs
sudo journalctl -u openvpn -f

# Monitor resource usage
top -p $(pgrep openvpn)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/openvpn"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/openvpn-backup-$DATE.tar.gz" /etc/openvpn /var/lib/openvpn

echo "Backup completed: $BACKUP_DIR/openvpn-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop openvpn

# Restore from backup
tar -xzf /backup/openvpn/openvpn-backup-*.tar.gz -C /

# Start service
sudo systemctl start openvpn
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u openvpn -n 100
sudo tail -f /var/log/openvpn/openvpn.log

# Check configuration
openvpn --version

# Check permissions
ls -la /etc/openvpn
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1194

# Test connectivity
telnet localhost 1194

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep openvpn)

# Check disk I/O
iotop -p $(pgrep openvpn)

# Check connections
ss -an | grep 1194
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  openvpn:
    image: openvpn:latest
    ports:
      - "1194:1194"
    volumes:
      - ./config:/etc/openvpn
      - ./data:/var/lib/openvpn
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update openvpn

# Debian/Ubuntu
sudo apt update && sudo apt upgrade openvpn

# Arch Linux
sudo pacman -Syu openvpn

# Alpine Linux
apk update && apk upgrade openvpn

# openSUSE
sudo zypper update openvpn

# FreeBSD
pkg update && pkg upgrade openvpn

# Always backup before updates
tar -czf /backup/openvpn-pre-update-$(date +%Y%m%d).tar.gz /etc/openvpn

# Restart after updates
sudo systemctl restart openvpn
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/openvpn

# Clean old logs
find /var/log/openvpn -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/openvpn
```

## Additional Resources

- Official Documentation: https://docs.openvpn.org/
- GitHub Repository: https://github.com/openvpn/openvpn
- Community Forum: https://forum.openvpn.org/
- Best Practices Guide: https://docs.openvpn.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
