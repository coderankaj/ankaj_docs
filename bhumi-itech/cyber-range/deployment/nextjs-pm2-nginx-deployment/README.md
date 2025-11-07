# Next.js Application Deployment with PM2 and Nginx

Complete guide for deploying Next.js applications using PM2 for process management and Nginx as a reverse proxy with SSL.

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Clone Repository](#step-1-clone-repository)
- [Step 2: Install Dependencies and Build](#step-2-install-dependencies-and-build)
- [Step 3: Run with PM2](#step-3-run-with-pm2)
- [Step 4: Configure Nginx](#step-4-configure-nginx)
- [Step 5: Setup SSL with Let's Encrypt](#step-5-setup-ssl-with-lets-encrypt)
- [Step 6: Verification](#step-6-verification)
- [PM2 Management](#pm2-management)
- [Application Updates](#application-updates)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Overview

This guide covers deploying a Next.js application (specifically the CyberRange Super User Dashboard) using:
- **Next.js**: React framework for production
- **PM2**: Process manager for Node.js applications
- **Nginx**: Reverse proxy and web server
- **Let's Encrypt**: Free SSL/TLS certificates

The dashboard provides a powerful interface for managing and monitoring your CyberRange environment.

---

## Prerequisites

- Ubuntu server (or similar Linux distribution)
- Node.js and npm installed (v14 or higher recommended)
- PM2 installed globally
- Nginx installed and configured
- Domain name pointing to your server's IP address
- Root or sudo access
- Certbot installed (for SSL)

### Install Prerequisites

```bash
# Install Node.js (if not already installed)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2 globally
sudo npm install -g pm2

# Install Nginx
sudo apt-get update
sudo apt-get install -y nginx

# Install Certbot for Nginx
sudo apt-get install -y python3-certbot-nginx
```

---

## Step 1: Clone Repository

Start by cloning the dashboard project from GitHub:

```bash
git clone https://github.com/coderankaj/cyberrange_dashboard_nextjs.git
cd cyberrange_dashboard_nextjs
```

**Note**: Replace the repository URL with your actual Next.js application repository if different.

---

## Step 2: Install Dependencies and Build

### Install Dependencies

```bash
npm i
```

Or if using yarn:

```bash
yarn install
```

### Build the Application

```bash
npm run build
```

For verbose build output (useful for debugging):

```bash
npm run build --verbose
```

**Note**: Ensure the build completes successfully without errors before proceeding.

---

## Step 3: Run with PM2

### Default Port (3000)

Start the application using PM2:

```bash
pm2 start npm --name cyberrange_dashboard_nextjs -- start
pm2 save
```

The `pm2 save` command saves the current process list so it will restart on system reboot.

### Custom Port

If you want the dashboard to run on a specific port (e.g., 3001):

```bash
PORT=3001 pm2 start npm --name cyberrange_dashboard_nextjs -- start
pm2 save
```

**Important**: If you change the port, remember to update the Nginx configuration accordingly (see Step 4).

### Verify PM2 Status

```bash
pm2 status
```

You should see `cyberrange_dashboard_nextjs` running with status "online".

---

## Step 4: Configure Nginx

### Create Nginx Configuration File

Create a new Nginx site configuration file:

```bash
sudo nano /etc/nginx/sites-available/cyberrange_dashboard_nextjs
```

### Add HTTP Configuration

Add the following configuration for initial HTTP access:

```nginx
server {
    listen 80;
    server_name dashboard.bhumiitech.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Configuration Notes**:
- Replace `dashboard.bhumiitech.com` with your actual domain name
- Replace `http://localhost:3000` with your custom port if you changed it (e.g., `http://localhost:3001`)
- The `Upgrade` and `Connection` headers enable WebSocket support
- `proxy_cache_bypass` ensures real-time updates work correctly

### Enable the Site

Create a symbolic link to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/cyberrange_dashboard_nextjs /etc/nginx/sites-enabled/
```

### Test and Reload Nginx

```bash
# Test Nginx configuration for syntax errors
sudo nginx -t

# If test passes, reload Nginx
sudo systemctl reload nginx
```

---

## Step 5: Setup SSL with Let's Encrypt

### Generate SSL Certificate

Use Certbot to generate and install a free SSL certificate:

```bash
sudo certbot --nginx -d dashboard.bhumiitech.com
```

Replace `dashboard.bhumiitech.com` with your actual domain name.

### Verify SSL Configuration

Certbot will automatically update your Nginx configuration. After setup, your config should look like this:

```nginx
server {
    server_name dashboard.bhumiitech.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/dashboard.bhumiitech.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/dashboard.bhumiitech.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = dashboard.bhumiitech.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name dashboard.bhumiitech.com;
    return 404; # managed by Certbot
}
```

**What Certbot Does**:
- Adds SSL certificate paths
- Configures HTTPS on port 443
- Redirects HTTP (port 80) to HTTPS
- Sets up secure SSL protocols and ciphers

For more details on SSL setup, see [SSL Certificate Setup Guide](../../../../commands/ssl-certbot/README.md).

---

## Step 6: Verification

### Check Application is Running

```bash
# Check PM2 status
pm2 status

# Check if port is listening
sudo netstat -tlnp | grep 3000
# Or
sudo ss -tlnp | grep 3000
```

### Test HTTP Access

Visit `http://dashboard.bhumiitech.com` in your browser. You should be redirected to HTTPS.

### Test HTTPS Access

Visit `https://dashboard.bhumiitech.com` in your browser.

**Verify**:
- ✅ The dashboard loads correctly
- ✅ HTTPS is working (green lock icon in browser)
- ✅ No mixed content warnings
- ✅ All resources load over HTTPS

---

## PM2 Management

### Basic Commands

```bash
# Check status of all processes
pm2 status

# View detailed information
pm2 show cyberrange_dashboard_nextjs

# View logs (real-time)
pm2 logs cyberrange_dashboard_nextjs

# View logs (last 100 lines)
pm2 logs cyberrange_dashboard_nextjs --lines 100

# Restart application
pm2 restart cyberrange_dashboard_nextjs

# Stop application
pm2 stop cyberrange_dashboard_nextjs

# Delete application from PM2
pm2 delete cyberrange_dashboard_nextjs

# Monitor resources (CPU, memory)
pm2 monit

# Save current process list
pm2 save

# List all processes
pm2 list
```

### Advanced PM2 Features

```bash
# Start with specific number of instances (cluster mode)
pm2 start npm --name cyberrange_dashboard_nextjs -i 4 -- start

# Set memory limit
pm2 start npm --name cyberrange_dashboard_nextjs --max-memory-restart 500M -- start

# Enable auto-restart on file changes (development)
pm2 start npm --name cyberrange_dashboard_nextjs --watch -- start

# View process information
pm2 info cyberrange_dashboard_nextjs

# Flush all logs
pm2 flush

# Reload application (zero-downtime)
pm2 reload cyberrange_dashboard_nextjs
```

### PM2 Startup Script

To ensure PM2 starts on system boot:

```bash
# Generate startup script
pm2 startup

# Follow the instructions provided, then save
pm2 save
```

---

## Application Updates

When updating the application:

### Step 1: Navigate to Project Directory

```bash
cd /path/to/cyberrange_dashboard_nextjs
```

### Step 2: Pull Latest Changes

```bash
git pull origin main
# Or your branch name
```

### Step 3: Install New Dependencies

```bash
npm i
```

### Step 4: Rebuild the Application

```bash
npm run build
```

### Step 5: Restart with PM2

```bash
pm2 restart cyberrange_dashboard_nextjs
```

### Step 6: Verify

```bash
# Check status
pm2 status

# Check logs for errors
pm2 logs cyberrange_dashboard_nextjs --lines 50
```

---

## Troubleshooting

### Application Not Starting

**Problem**: PM2 shows application as "errored" or "stopped"

**Solutions**:
1. Check PM2 logs:
   ```bash
   pm2 logs cyberrange_dashboard_nextjs
   ```

2. Verify Node.js version:
   ```bash
   node --version
   npm --version
   ```

3. Check if port is already in use:
   ```bash
   sudo lsof -i :3000
   # Or
   sudo netstat -tlnp | grep 3000
   ```

4. Verify build was successful:
   ```bash
   npm run build
   ```

### Nginx 502 Bad Gateway

**Problem**: Browser shows "502 Bad Gateway" error

**Solutions**:
1. Verify PM2 is running:
   ```bash
   pm2 status
   ```

2. Check if the application is listening on the correct port:
   ```bash
   sudo netstat -tlnp | grep 3000
   ```

3. Verify Nginx proxy_pass URL matches PM2 port:
   ```bash
   sudo nano /etc/nginx/sites-available/cyberrange_dashboard_nextjs
   # Check proxy_pass matches your PM2 port
   ```

4. Check Nginx error logs:
   ```bash
   sudo tail -f /var/log/nginx/error.log
   ```

### Port Conflicts

**Problem**: Port 3000 (or custom port) is already in use

**Solutions**:
1. Find the process using the port:
   ```bash
   sudo lsof -i :3000
   ```

2. Kill the process (replace PID):
   ```bash
   sudo kill -9 PID
   ```

3. Or use a different port:
   ```bash
   PORT=3001 pm2 start npm --name cyberrange_dashboard_nextjs -- start
   ```
   Then update Nginx configuration accordingly.

### Build Errors

**Problem**: `npm run build` fails

**Solutions**:
1. Clear node_modules and reinstall:
   ```bash
   rm -rf node_modules package-lock.json
   npm install
   npm run build
   ```

2. Check Node.js version compatibility:
   ```bash
   node --version
   # Next.js typically requires Node.js 14.x or higher
   ```

3. Review build logs for specific errors:
   ```bash
   npm run build --verbose
   ```

### SSL Certificate Issues

**Problem**: SSL certificate errors or HTTPS not working

**Solutions**:
1. Test certificate renewal:
   ```bash
   sudo certbot renew --dry-run
   ```

2. Check certificate expiration:
   ```bash
   sudo certbot certificates
   ```

3. Manually renew certificate:
   ```bash
   sudo certbot renew
   sudo systemctl reload nginx
   ```

For more SSL troubleshooting, see [SSL Certificate Setup Guide](../../../../commands/ssl-certbot/README.md).

### Application Crashes

**Problem**: Application keeps crashing or restarting

**Solutions**:
1. Check PM2 logs for errors:
   ```bash
   pm2 logs cyberrange_dashboard_nextjs --err
   ```

2. Check system resources:
   ```bash
   pm2 monit
   # Or
   htop
   ```

3. Increase memory limit if needed:
   ```bash
   pm2 delete cyberrange_dashboard_nextjs
   pm2 start npm --name cyberrange_dashboard_nextjs --max-memory-restart 1G -- start
   pm2 save
   ```

---

## Best Practices

1. **Environment Variables**: Use `.env` files for configuration (never commit secrets)
2. **Process Management**: Always use PM2 for production deployments
3. **Auto-Restart**: Enable PM2 startup script for automatic restart on reboot
4. **Monitoring**: Set up monitoring for PM2 processes and application health
5. **Logs**: Regularly check and rotate logs to prevent disk space issues
6. **Updates**: Test updates in staging before deploying to production
7. **Backups**: Regularly backup your application code and configuration
8. **SSL**: Always use HTTPS in production (Let's Encrypt provides free certificates)
9. **Security**: Keep Node.js, npm, and system packages updated
10. **Resource Limits**: Set appropriate memory limits in PM2 to prevent memory leaks

---

## Quick Reference

### Complete Deployment Commands

```bash
# 1. Clone and setup
git clone https://github.com/coderankaj/cyberrange_dashboard_nextjs.git
cd cyberrange_dashboard_nextjs
npm i
npm run build

# 2. Start with PM2
pm2 start npm --name cyberrange_dashboard_nextjs -- start
pm2 save

# 3. Configure Nginx
sudo nano /etc/nginx/sites-available/cyberrange_dashboard_nextjs
# Add configuration (see Step 4)
sudo ln -s /etc/nginx/sites-available/cyberrange_dashboard_nextjs /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# 4. Setup SSL
sudo certbot --nginx -d dashboard.bhumiitech.com

# 5. Verify
pm2 status
curl https://dashboard.bhumiitech.com
```

### Common PM2 Commands

```bash
pm2 status                    # Check status
pm2 logs cyberrange_dashboard_nextjs  # View logs
pm2 restart cyberrange_dashboard_nextjs  # Restart
pm2 stop cyberrange_dashboard_nextjs  # Stop
pm2 delete cyberrange_dashboard_nextjs  # Remove
pm2 monit                     # Monitor resources
pm2 save                      # Save process list
```

---

## Related Documentation

- [SSL Certificate Setup Guide](../../../../commands/ssl-certbot/README.md)
- [Commands Reference](../../../../commands/README.md)
- [CyberRange Deployment Guide](../README.md)

---

*Last updated: 2024*

