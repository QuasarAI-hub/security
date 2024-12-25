# Server Security Guide

Welcome to the Server Security Guide. This comprehensive guide will help you secure your validator node and maintain high security standards in the Cosmos ecosystem.

## Table of Contents
- [Quick Start](#quick-start)
- [Initial Server Setup](#initial-server-setup)
- [SSH Security](#ssh-security)
- [Firewall Configuration](#firewall-configuration)
- [Fail2ban Setup](#fail2ban-setup)
- [System Hardening](#system-hardening)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)

## Quick Start

### Minimum Security Checklist
✅ Create non-root user
✅ Configure SSH key authentication
✅ Disable password authentication
✅ Set up UFW firewall
✅ Install and configure Fail2ban
✅ Regular security updates
✅ Enable system monitoring

## Initial Server Setup

### Create Admin User
```bash
# Add new user
sudo adduser val

# Add to sudo group
sudo usermod -aG sudo val

# Switch to new user
su - val
```

### System Updates
```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Install essential security tools
sudo apt install ufw fail2ban unattended-upgrades
```

## SSH Security

### Generate SSH Keys (On Local Machine)
```bash
# Generate Ed25519 key (more secure than RSA)
ssh-keygen -t ed25519 -C "validator@cosmos"

# Copy key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub val@SERVER_IP
```

### Secure SSH Configuration
Edit `/etc/ssh/sshd_config`:
```bash
# Disable root login
PermitRootLogin no

# Disable password authentication
PasswordAuthentication no

# Allow only specific user
AllowUsers admin

# Use specific SSH version
Protocol 2

# Set stricter security options
MaxAuthTries 3
PubkeyAuthentication yes
AuthenticationMethods publickey

```

Restart SSH service:
```bash
sudo systemctl restart sshd
```

## Firewall Configuration

### UFW Setup
```bash
# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (use your custom port)
sudo ufw allow 22/tcp

# Allow validator ports (example for Cosmos)
sudo ufw allow 26656/tcp  # P2P
sudo ufw allow 26657/tcp  # RPC (restrict to trusted IPs)
sudo ufw allow 26660/tcp  # Prometheus metrics

# Enable firewall
sudo ufw enable
```

### Advanced UFW Rules
```bash
# Allow specific IP addresses
sudo ufw allow from TRUSTED_IP to any port 26657

# Rate limiting
sudo ufw limit 22/tcp

# Logging
sudo ufw logging on
```

## Fail2ban Setup

### Basic Configuration
Create `/etc/fail2ban/jail.local`:
```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```

### Custom Fail2ban Filter
Create `/etc/fail2ban/filter.d/cosmos-dos.conf`:
```ini
[Definition]
failregex = ^ Blacklisted IP tried to connect: <HOST>
ignoreregex =
```

Restart Fail2ban:
```bash
sudo systemctl restart fail2ban
```

## System Hardening

### Secure Shared Memory
Add to `/etc/fstab`:
```
tmpfs     /run/shm     tmpfs     defaults,noexec,nosuid     0     0
```

### System Limits
Edit `/etc/security/limits.conf`:
```
* soft nofile 65535
* hard nofile 65535
```

### Secure Kernel Parameters
Edit `/etc/sysctl.conf`:
```ini
# Network security
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Prevent IP spoofing
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Load new settings
sudo sysctl -p
```

## Monitoring

### Essential Monitoring Tools
```bash
# Install monitoring tools
sudo apt install htop iftop nethogs tmux

# Set up Prometheus metrics
cosmos_exporter:
  enabled: true
  listen_addr: "127.0.0.1:26660"
```

### Log Monitoring
```bash
# Configure logrotate
sudo nano /etc/logrotate.d/cosmos
```
```ini
/var/log/cosmos/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 cosmos cosmos
    sharedscripts
    postrotate
        systemctl reload cosmos > /dev/null 2>&1 || true
    endscript
}
```

## Troubleshooting

### Common Issues

#### SSH Access Denied
1. Check SSH key permissions:
```bash
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

2. Verify SSH configuration:
```bash
sudo sshd -T
```

#### Firewall Issues
1. Check UFW status:
```bash
sudo ufw status verbose
```

2. Review logs:
```bash
sudo tail -f /var/log/ufw.log
```

#### Fail2ban Troubleshooting
1. Check banned IPs:
```bash
sudo fail2ban-client status sshd
```

2. Unban IP:
```bash
sudo fail2ban-client set sshd unbanip IP_ADDRESS
```

## Best Practices

### Regular Maintenance
✅ Update systems weekly
✅ Review logs daily
✅ Backup configuration files
✅ Test security measures
✅ Monitor system resources
✅ Rotate access credentials

### Security Auditing
1. Check for unauthorized access
2. Review system logs
3. Monitor resource usage
4. Verify network connections
5. Test backup procedures

## Emergency Response

### If You Detect a Breach
1. Isolate the server
```bash
sudo ufw deny all incoming
```

2. Preserve evidence
```bash
sudo cp /var/log/* /secure-backup/
```

3. Contact support
- Security Team: [@whtech_support](https://t.me/whtech_support)
- Emergency Line: [Discord](https://discord.gg/tZW4xf3c2D)

## Need Help?
- Technical Support: [@whtech_support](https://t.me/whtech_support)
- Community: [Telegram](https://t.me/quasarstakingeng)
- Documentation: [Website](https://quasarstaking.ai)

---

*Maintained by Quasar - Securing the Cosmos Ecosystem*

⚠️ Remember: Server security is an ongoing process. Regular updates and monitoring are essential!
