# Dendrite Installation Guide

Dendrite is a free and open-source Matrix Server. Second-generation Matrix homeserver

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 8008/8448 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8008/8448 (default dendrite port)
- **Dependencies**:
  - postgresql
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

# Install dendrite
sudo dnf install -y dendrite postgresql

# Enable and start service
sudo systemctl enable --now dendrite

# Configure firewall
sudo firewall-cmd --permanent --add-service=dendrite
sudo firewall-cmd --reload

# Verify installation
dendrite --version || systemctl status dendrite
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install dendrite
sudo apt install -y dendrite postgresql

# Enable and start service
sudo systemctl enable --now dendrite

# Configure firewall
sudo ufw allow 8008/8448

# Verify installation
dendrite --version || systemctl status dendrite
```

### Arch Linux

```bash
# Install dendrite
sudo pacman -S dendrite

# Enable and start service
sudo systemctl enable --now dendrite

# Verify installation
dendrite --version || systemctl status dendrite
```

### Alpine Linux

```bash
# Install dendrite
apk add --no-cache dendrite

# Enable and start service
rc-update add dendrite default
rc-service dendrite start

# Verify installation
dendrite --version || rc-service dendrite status
```

### openSUSE/SLES

```bash
# Install dendrite
sudo zypper install -y dendrite postgresql

# Enable and start service
sudo systemctl enable --now dendrite

# Configure firewall
sudo firewall-cmd --permanent --add-service=dendrite
sudo firewall-cmd --reload

# Verify installation
dendrite --version || systemctl status dendrite
```

### macOS

```bash
# Using Homebrew
brew install dendrite

# Start service
brew services start dendrite

# Verify installation
dendrite --version
```

### FreeBSD

```bash
# Using pkg
pkg install dendrite

# Enable in rc.conf
echo 'dendrite_enable="YES"' >> /etc/rc.conf

# Start service
service dendrite start

# Verify installation
dendrite --version || service dendrite status
```

### Windows

```powershell
# Using Chocolatey
choco install dendrite

# Or using Scoop
scoop install dendrite

# Verify installation
dendrite --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/dendrite

# Set up basic configuration
sudo tee /etc/dendrite/dendrite.conf << 'EOF'
# Dendrite Configuration
max_open_conns: 100
EOF

# Test configuration
sudo dendrite -t || sudo dendrite configtest

# Reload service
sudo systemctl reload dendrite
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R dendrite:dendrite /etc/dendrite
sudo chmod 750 /etc/dendrite

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable dendrite

# Start service
sudo systemctl start dendrite

# Stop service
sudo systemctl stop dendrite

# Restart service
sudo systemctl restart dendrite

# Reload configuration
sudo systemctl reload dendrite

# Check status
sudo systemctl status dendrite

# View logs
sudo journalctl -u dendrite -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add dendrite default

# Start service
rc-service dendrite start

# Stop service
rc-service dendrite stop

# Restart service
rc-service dendrite restart

# Check status
rc-service dendrite status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'dendrite_enable="YES"' >> /etc/rc.conf

# Start service
service dendrite start

# Stop service
service dendrite stop

# Restart service
service dendrite restart

# Check status
service dendrite status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start dendrite
brew services stop dendrite
brew services restart dendrite

# Check status
brew services list | grep dendrite
```

### Windows Service Manager

```powershell
# Start service
net start dendrite

# Stop service
net stop dendrite

# Using PowerShell
Start-Service dendrite
Stop-Service dendrite
Restart-Service dendrite

# Check status
Get-Service dendrite
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/dendrite/dendrite.conf << 'EOF'
max_open_conns: 100
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart dendrite
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream dendrite_backend {
    server 127.0.0.1:8008/8448;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name dendrite.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name dendrite.example.com;

    ssl_certificate /etc/ssl/certs/dendrite.example.com.crt;
    ssl_certificate_key /etc/ssl/private/dendrite.example.com.key;

    location / {
        proxy_pass http://dendrite_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName dendrite.example.com
    Redirect permanent / https://dendrite.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName dendrite.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/dendrite.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/dendrite.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8008/8448/
    ProxyPassReverse / http://127.0.0.1:8008/8448/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:8008/8448/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend dendrite_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/dendrite.pem
    redirect scheme https if !{ ssl_fc }
    default_backend dendrite_backend

backend dendrite_backend
    balance roundrobin
    option httpchk GET /health
    server dendrite1 127.0.0.1:8008/8448 check
    server dendrite2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R dendrite:dendrite /etc/dendrite
sudo chmod 750 /etc/dendrite

# Configure firewall
sudo firewall-cmd --permanent --add-service=dendrite
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/dendrite.conf << 'EOF'
[dendrite]
enabled = true
port = 8008/8448
filter = dendrite
logpath = /var/log/dendrite/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/dendrite.key \
    -out /etc/ssl/certs/dendrite.crt

# Configure SSL in dendrite
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE dendrite_db;
CREATE USER dendrite_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE dendrite_db TO dendrite_user;
EOF

# Configure dendrite to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE dendrite_db;
CREATE USER 'dendrite_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON dendrite_db.* TO 'dendrite_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Dendrite specific tuning
max_open_conns: 100
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
dendrite soft nofile 65535
dendrite hard nofile 65535
dendrite soft nproc 32768
dendrite hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'dendrite'
    static_configs:
      - targets: ['localhost:8008/8448']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet dendrite; then
    echo "Dendrite is running"
    exit 0
else
    echo "Dendrite is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/dendrite << 'EOF'
/var/log/dendrite/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 dendrite dendrite
    postrotate
        systemctl reload dendrite > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Dendrite backup script
BACKUP_DIR="/backup/dendrite"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop dendrite

# Backup configuration
tar -czf "$BACKUP_DIR/dendrite-config-$DATE.tar.gz" /etc/dendrite

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/dendrite-data-$DATE.tar.gz" /var/lib/dendrite

# Start service
systemctl start dendrite

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop dendrite

# Restore configuration
sudo tar -xzf /backup/dendrite/dendrite-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/dendrite/dendrite-data-*.tar.gz -C /

# Set permissions
sudo chown -R dendrite:dendrite /etc/dendrite
sudo chown -R dendrite:dendrite /var/lib/dendrite

# Start service
sudo systemctl start dendrite
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u dendrite -n 100
sudo tail -f /var/log/dendrite/*.log

# Check configuration
sudo dendrite -t || sudo dendrite configtest

# Check permissions
ls -la /etc/dendrite
ls -la /var/lib/dendrite
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8008/8448
sudo netstat -tlnp | grep 8008/8448

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 8008/8448
nc -zv localhost 8008/8448
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep dendrite)
htop -p $(pgrep dendrite)

# Check connections
ss -ant | grep :8008/8448 | wc -l

# Monitor I/O
iotop -p $(pgrep dendrite)
```

### Debug Mode

```bash
# Run in debug mode
sudo dendrite -d
# or
sudo dendrite debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  dendrite:
    image: dendrite:latest
    container_name: dendrite
    ports:
      - "8008/8448:8008/8448"
    volumes:
      - ./config:/etc/dendrite
      - ./data:/var/lib/dendrite
    environment:
      - dendrite_CONFIG=/etc/dendrite/dendrite.conf
    restart: unless-stopped
    networks:
      - dendrite_net

networks:
  dendrite_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dendrite
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dendrite
  template:
    metadata:
      labels:
        app: dendrite
    spec:
      containers:
      - name: dendrite
        image: dendrite:latest
        ports:
        - containerPort: 8008/8448
        volumeMounts:
        - name: config
          mountPath: /etc/dendrite
      volumes:
      - name: config
        configMap:
          name: dendrite-config
---
apiVersion: v1
kind: Service
metadata:
  name: dendrite
spec:
  selector:
    app: dendrite
  ports:
  - port: 8008/8448
    targetPort: 8008/8448
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Dendrite
  hosts: all
  become: yes
  tasks:
    - name: Install dendrite
      package:
        name: dendrite
        state: present
    
    - name: Configure dendrite
      template:
        src: dendrite.conf.j2
        dest: /etc/dendrite/dendrite.conf
        owner: dendrite
        group: dendrite
        mode: '0640'
      notify: restart dendrite
    
    - name: Start and enable dendrite
      systemd:
        name: dendrite
        state: started
        enabled: yes
  
  handlers:
    - name: restart dendrite
      systemd:
        name: dendrite
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update dendrite

# Debian/Ubuntu
sudo apt update && sudo apt upgrade dendrite

# Arch Linux
sudo pacman -Syu dendrite

# Alpine Linux
apk update && apk upgrade dendrite

# openSUSE
sudo zypper update dendrite

# FreeBSD
pkg update && pkg upgrade dendrite

# Always backup before updates
tar -czf /backup/dendrite-pre-update-$(date +%Y%m%d).tar.gz /etc/dendrite

# Restart after updates
sudo systemctl restart dendrite
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/dendrite -name "*.log" -mtime +30 -delete

# Verify integrity
sudo dendrite --verify || sudo dendrite check

# Update databases (if applicable)
sudo dendrite-update-db

# Optimize performance
sudo dendrite-optimize

# Check for security updates
sudo dendrite --security-check
```

## Additional Resources

- Official Documentation: https://docs.dendrite.org/
- GitHub Repository: https://github.com/dendrite/dendrite
- Community Forum: https://forum.dendrite.org/
- Wiki: https://wiki.dendrite.org/
- Comparison vs Synapse, Conduit, Construct: https://docs.dendrite.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
