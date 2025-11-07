# Cyber Range Platform - Deployment Guide

Complete deployment guide for the Cyber Range platform components.

## 📋 Table of Contents

- [Backend Deployment](#backend-deployment)
- [MongoDB Setup](#mongodb-setup)
- [Frontend Deployment](#frontend-deployment)
- [Super User Dashboard](#super-user-dashboard)
- [OpenStack Configuration](#openstack-configuration)
- [Troubleshooting](#troubleshooting)

---

## Backend Deployment

For complete backend deployment instructions, see the [Backend Deployment Guide](backend/README.md).

### Quick Overview

The backend consists of:
- **Django Application**: Main API server
- **Daphne**: ASGI server for Django
- **Celery Worker**: Background task processing
- **Celery Beat**: Scheduled task scheduler
- **Redis**: Message broker for Celery

### Key Steps

1. Setup virtual environment and install dependencies
2. Configure environment variables (`.env` file)
3. Setup Redis
4. Configure systemd services
5. Configure Nginx reverse proxy

📖 **Full Guide**: [Backend Deployment Guide](backend/README.md)

---

## MongoDB Setup

For complete MongoDB installation and configuration, see the [MongoDB Setup Guide](mongodb/README.md).

### Quick Overview

MongoDB is used as the primary database for the Cyber Range platform.

### Key Steps

1. Install MongoDB
2. Configure security settings
3. Create database users
4. Configure connection strings

📖 **Full Guide**: [MongoDB Setup Guide](mongodb/README.md)

---

## Frontend Deployment

### Prerequisites

- Node.js and npm
- Build tools

### Step 1: Build the Application

#### Development Build

```bash
GENERATE_SOURCEMAP=false NODE_OPTIONS="--max-old-space-size=8192" npm run build:dev
```

#### Production Build

```bash
# For 4GB memory
NODE_OPTIONS="--max-old-space-size=4096" npm run build:prod

# For 8GB memory
NODE_OPTIONS="--max-old-space-size=8192" npm run build:prod
```

### Step 2: Deploy to Server

The frontend code is located at:
```
/home/ubuntu/rangestorm2.5/cyber_range_frontend_v2.5
```

### Step 3: Deploy to Web Root

```bash
# 1. Create a timestamped backup of current /var/www/html
sudo mv /var/www/html /var/www/html.bak-$(date +%Y%m%d-%H%M%S)

# 2. Copy build contents to /var/www/html (including hidden files)
sudo cp -r ~/rangestorm2.5/cyber_range_frontend_v2.5/build/. /var/www/html

# 3. Set correct ownership and permissions
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html

# 4. Test Nginx configuration
sudo nginx -t

# 5. Restart Nginx
sudo systemctl restart nginx

# 6. Enable Nginx on boot (if not already enabled)
sudo systemctl enable nginx
```


## Super User Dashboard

### Prerequisites

- Node.js and npm
- PM2
- Nginx

### Step 1: Clone Repository

```bash
git clone https://github.com/coderankaj/cyberrange_dashboard_nextjs.git
cd cyberrange_dashboard_nextjs
```

### Step 2: Install Dependencies and Build

```bash
npm i
npm run build
```

For verbose build output:

```bash
npm run build --verbose
```

### Step 3: Run with PM2

#### Default Port (3000)

```bash
pm2 start npm --name cyberrange_dashboard_nextjs -- start
pm2 save
```

#### Custom Port (e.g., 3001)

```bash
PORT=3001 pm2 start npm --name cyberrange_dashboard_nextjs -- start
pm2 save
```

**Note**: Update Nginx configuration if using a custom port.

### Step 4: Configure Nginx

Create Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/cyberrange_dashboard_nextjs
```

Add the following configuration:

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

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/cyberrange_dashboard_nextjs /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Step 5: Setup SSL with Certbot

```bash
sudo certbot --nginx -d dashboard.bhumiitech.com
```

Certbot will automatically update your Nginx configuration with SSL settings.

### Step 6: Verify Deployment

Visit `https://dashboard.bhumiitech.com` in your browser.

### PM2 Management Commands

```bash
# Check status
pm2 status

# Restart dashboard
pm2 restart cyberrange_dashboard_nextjs

# View logs
pm2 logs cyberrange_dashboard_nextjs
```

---

## OpenStack Configuration

### Network/Scenario Creation Updates

**Important**: If scenario instances are not being created, update the following:

#### 1. Update Router Name

Navigate to the cloud management directory:

```bash
cd cloud_management
nano utils.py
```

Find and update the `connect_router_to_public_network` function:

```python
def connect_router_to_public_network(router, public_network_name="Public-Lan"):
    # Change "Public-Lan" to your actual public network name
```

#### 2. Update Availability Zone

Find and update the `create_cloud_instance` function:

```python
def create_cloud_instance(instance_name, instance_image_id, instance_flavor_id, instance_network_id, instance_availability_zone="Compute1.bhumiitech.com"):
    # Change "Compute1.bhumiitech.com" to your actual availability zone
```


## Troubleshooting

### Backend Issues

For detailed backend troubleshooting, see the [Backend Deployment Guide](backend/README.md#troubleshooting).

Common issues:
- Import module errors
- Service not starting
- Port conflicts
- Redis connection issues

### MongoDB Issues

For detailed MongoDB troubleshooting, see the [MongoDB Setup Guide](mongodb/README.md#troubleshooting).

Common issues:
- Connection refused
- Authentication failed
- Permission denied

### Frontend Build Issues

#### Memory Errors

Increase Node.js memory limit:

```bash
NODE_OPTIONS="--max-old-space-size=8192" npm run build:prod
```

### OpenStack Instance Creation Issues

1. Verify router name in `cloud_management/utils.py`
2. Check availability zone configuration
3. Verify OpenStack credentials in `.env` file

---

*For additional help, refer to the main [Bhumi Itech Documentation](../README.md)*

