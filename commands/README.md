# Commands Reference

Quick reference guide for commonly used commands.

## Git Commands

### Reset and Clean Working Directory

```bash
# Hard reset to HEAD (discards all local changes)
git reset --hard HEAD
```

### Stash Management

```bash
# Stash all files
git stash

# Stash selected files with a message
git stash push -m "temp local changes" blacklisted_domains.txt cloud_management/utils.py cyber_range_platform/settings.py requirements.txt

# Pull latest changes
git pull

# Restore stashed changes
git stash pop
```

## SSL Certificate Setup (Certbot)

For complete SSL certificate installation and management guide, see [SSL Certificate Setup with Let's Encrypt (Certbot)](ssl-certbot/README.md).

### Quick Reference

```bash
# Generate SSL certificate for Nginx
sudo certbot --nginx -d dashboard.bhumiitech.com

# Generate SSL certificate for Apache
sudo certbot --apache -d yourdomain.com

# Check certificate status
sudo certbot certificates

# Test renewal
sudo certbot renew --dry-run
```

📖 **Full Guide**: [SSL Certificate Setup Guide](ssl-certbot/README.md)

## Terminal History

### Clear Terminal History

```bash
# Clear current session history
history -c && history -w

# Remove bash history file
rm ~/.bash_history

# Remove zsh history file
rm ~/.zsh_history
```

## WiFi Management

### WiFi Control

```bash
# Turn WiFi off
sudo nmcli radio wifi off

# Turn WiFi on
sudo nmcli radio wifi on

# List available WiFi networks
sudo nmcli device wifi list

# Connect to WiFi network
sudo nmcli device wifi connect "<SSID>" password "<password>"

# Rescan for networks
sudo nmcli device wifi rescan
sleep 2
sudo nmcli device wifi list
```

### Example WiFi Connections

```bash
# Connect to specific networks
sudo nmcli device wifi connect "Airtel_hari_4052" password "air04786"
sudo nmcli device wifi connect "narzo" password "@ankaj11"
sudo nmcli device wifi connect AA:6A:B1:A7:6A:A6 password '@ankaj11'
sudo nmcli device wifi connect "Airtel_gane_4024" password 'air55956'
```

## Next.js Deployment with PM2 and Nginx

For complete guide on deploying Next.js applications with PM2 and Nginx, see [Next.js Application Deployment Guide](../bhumi-itech/cyber-range/deployment/nextjs-pm2-nginx-deployment/README.md).

### Quick Reference

```bash
# Clone and build
git clone <repository-url>
cd <project-directory>
npm i
npm run build

# Start with PM2
pm2 start npm --name app-name -- start
pm2 save

# Setup SSL
sudo certbot --nginx -d yourdomain.com
```

📖 **Full Guide**: [Next.js Deployment Guide](../bhumi-itech/cyber-range/deployment/nextjs-pm2-nginx-deployment/README.md)

---

*For project-specific commands, see [Bhumi Itech Documentation](../bhumi-itech/README.md)*

