# Backend Deployment Guide

Complete guide for deploying the Cyber Range platform backend services.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment Configuration](#environment-configuration)
- [Service Configuration](#service-configuration)
- [Nginx Configuration](#nginx-configuration)
- [Running Services](#running-services)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- Python 3.x
- Virtual environment
- Redis
- MongoDB (see [MongoDB Setup](../mongodb/README.md))

---

## Installation

### Step 1: Virtual Environment Setup

```bash
python3 -m venv venv
source venv/bin/activate  # On Linux/Mac
# or
venv\Scripts\activate  # On Windows
```

### Step 2: Install Dependencies

```bash
pip install -r requirements.txt
```

**Note**: If you encounter import module errors, run:
```bash
pip install --upgrade future
```

### Step 3: Redis Installation

```bash
sudo apt install redis
sudo systemctl enable redis
sudo systemctl start redis
```

Verify Redis is running:
```bash
redis-cli ping
# Should return: PONG
```

---

## Environment Configuration

Create a `.env` file in the backend root directory with the following variables:

```env
################################## Django Settings ##################################
SECRET_KEY = 'your-secret-key-here'
DEBUG = False

################################## Allowed Hosts ##################################
SSL_DOMAIN = 'cyberrangebackend1.bhumiitech.com'
PRIVATE_IP = '10.1.20.75'
LOCALHOST1 = 'www.cyberrangebackend1.bhumiitech.com'
LOCALHOST2 = 'http://localhost:3000'
LOCALHOST3 = 'dashboard.bhumiitech.com'

################################## Celery DB Settings ##################################
HOST = "mongodb://username:password@10.1.20.75:27017/"
DATABASE = "cyber_range"
TASKMETA_COLLECTION = "celery_taskmeta"

################################## Email Credentials ##################################
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_HOST_USER = 'support@bhumiitech.com'
EMAIL_HOST_PASSWORD = 'your-email-password'
EMAIL_USE_TLS = True

################################## MongoDB URI Strings ##################################
MONGO_DB_URI = "mongodb://username:password@10.1.20.75:27017/"

################################## Core Utility Constants ##################################
API_URL = 'https://cyberrangebackend1.bhumiitech.com'
FRONTEND_URL = 'https://cyberrange.bhumiitech.com'
EMAIL_LOGO_URL = "https://rangestormbackend.bhumiitech.com/static/images/logo/logo_2.png"
BLACKLISTED_DOMAINS_URL = "https://raw.githubusercontent.com/disposable-email-domains/disposable-email-domains/master/disposable_email_blocklist.conf"
WHITELISTED_DOMAINS_URL = ["gmail.com", "outlook.com", "protonmail.com", "bhumiitech.com", "microsoft.com", "proton.me"]

########### OpenStack Credentials ##################
AUTH_URL = 'http://10.1.75.40:5000'
PROJECT_ID = 'your-project-id'
USER_DOMAIN_NAME = 'Default'
USERNAME = 'your-username'
PASSWORD = 'your-password'

################### Token Lifetime ########
ACCESS_TOKEN_EXPIRATION_MINUTES = 300
REFRESH_TOKEN_EXPIRATION_MINUTES = 400

###### CRED ###############
USER_EMAIL = "support@bhumiitech.com"
USER_PASSWORD = "your-password"

###############################
FOUR_CORE_BASE_URL='https://prod.fourcore.io'
FOUR_CORE_API_KEY='your-api-key'
```

**Important**: Update all variables according to your setup.

---

## Service Configuration

### Systemd Service Files

#### Daphne Service (ASGI Server)

Location: `/etc/systemd/system/daphne.service`

```ini
[Unit]
Description=Daphne Service
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/rangestorm2.5/cyber_range_platform_v2
ExecStart=/home/ubuntu/rangestorm/cyber_range_platform_v2/venv/bin/daphne -b 0.0.0.0 -p 8000 cyber_range_platform.asgi:application
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

#### Celery Worker Service

Location: `/etc/systemd/system/celery-worker.service`

```ini
[Unit]
Description=Celery Worker
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/rangestorm2.5/cyber_range_platform_v2
ExecStart=/home/ubuntu/rangestorm/cyber_range_platform_v2/venv/bin/celery -A cyber_range_platform.celery worker --pool=solo -l info

[Install]
WantedBy=multi-user.target
```

#### Celery Beat Service (Scheduler)

Location: `/etc/systemd/system/celery-beat.service`

```ini
[Unit]
Description=Celery Beat Scheduler for Cyber Range Platform
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/rangestorm2.5/cyber_range_platform_v2
ExecStart=celery -A cyber_range_platform beat -l INFO
Restart=always

[Install]
WantedBy=multi-user.target
```

### Setting Up Services

After creating the service files:

```bash
# Reload systemd to recognize new services
sudo systemctl daemon-reload

# Enable services to start on boot
sudo systemctl enable daphne.service
sudo systemctl enable celery-worker.service
sudo systemctl enable celery-beat.service

# Start services
sudo systemctl start daphne.service
sudo systemctl start celery-worker.service
sudo systemctl start celery-beat.service
```

### Service Management Commands

```bash
# Check service status
sudo systemctl status daphne.service
sudo systemctl status celery-worker.service
sudo systemctl status celery-beat.service

# Restart services
sudo systemctl restart daphne.service
sudo systemctl restart celery-worker.service
sudo systemctl restart celery-beat.service

# View logs
journalctl -u daphne.service -f
journalctl -u celery-worker.service -f
journalctl -u celery-beat.service -f

# Stop services
sudo systemctl stop daphne.service
sudo systemctl stop celery-worker.service
sudo systemctl stop celery-beat.service
```

---

## Nginx Configuration

### Backend Nginx Configuration

Location: `/etc/nginx/sites-available/backend`

```nginx
server {
    server_name cyberrangebackend1.bhumiitech.com;

    location /static/ {
        alias /home/ubuntu/rangestorm2.5/cyber_range_platform_v2/static/;
        add_header Access-Control-Allow-Origin https://cyberrange.bhumiitech.com;
        add_header Cross-Origin-Resource-Policy cross-origin;
        add_header Access-Control-Allow-Methods "GET, OPTIONS";
        add_header Access-Control-Expose-Headers "Content-Length, Content-Range";
        autoindex on;
    }

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/cyberrangebackend1.bhumiitech.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cyberrangebackend1.bhumiitech.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = cyberrangebackend1.bhumiitech.com) {
        return 301 https://$host$request_uri;
    }

    server_name cyberrangebackend1.bhumiitech.com;
    listen 80;
    return 404;
}
```

### Enable Nginx Configuration

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### Setup SSL Certificate

```bash
sudo certbot --nginx -d cyberrangebackend1.bhumiitech.com
```

---

## Running Services

### Development Mode (Testing Only)

For testing purposes, you can run services manually:

```bash
# Activate virtual environment
source venv/bin/activate

# Celery Worker
celery -A cyber_range_platform.celery worker --pool=solo -l info > celery.log

# Celery Beat (Scheduler) - in a separate terminal
celery -A cyber_range_platform beat -l INFO

# Daphne (ASGI Server) - in a separate terminal
daphne -b 0.0.0.0 -p 8000 cyber_range_platform.asgi:application
```

**Note**: For production, always use systemd services as described in [Service Configuration](#service-configuration).

---

## Troubleshooting

### Import Module Errors

```bash
pip install --upgrade future
```

### Service Not Starting

1. Check service status:
   ```bash
   sudo systemctl status daphne.service
   ```

2. View detailed logs:
   ```bash
   journalctl -u daphne.service -f
   ```

3. Verify virtual environment path in service file matches actual path

4. Check file permissions:
   ```bash
   ls -la /home/ubuntu/rangestorm2.5/cyber_range_platform_v2/venv/bin/daphne
   ```

### Port Already in Use

If port 8000 is already in use:

```bash
# Find process using port 8000
sudo lsof -i :8000

# Kill the process (replace PID with actual process ID)
sudo kill -9 PID
```

### Redis Connection Issues

```bash
# Check if Redis is running
sudo systemctl status redis

# Test Redis connection
redis-cli ping
```

### MongoDB Connection Issues

1. Verify MongoDB is running (see [MongoDB Setup](../mongodb/README.md))
2. Check MongoDB credentials in `.env` file
3. Test MongoDB connection:
   ```bash
   mongosh "mongodb://username:password@10.1.20.75:27017/"
   ```

### Static Files Not Serving

1. Collect static files:
   ```bash
   python manage.py collectstatic
   ```

2. Verify static files directory exists and has correct permissions:
   ```bash
   ls -la /home/ubuntu/rangestorm2.5/cyber_range_platform_v2/static/
   ```

3. Check Nginx configuration for static files path

---

## Related Documentation

- [MongoDB Setup](../mongodb/README.md)
- [Frontend Deployment](../README.md#frontend-deployment)
- [OpenStack Configuration](../README.md#openstack-configuration)
- [Main Deployment Guide](../README.md)

---

*For additional help, refer to the main [Deployment Guide](../README.md)*

