# Backend Nginx Configuration

Nginx reverse proxy configuration for the Django backend application.

## Overview

This Nginx configuration serves as a reverse proxy for the Django backend, handling SSL termination, static file serving, and proxying requests to the Daphne ASGI server.

## Configuration File Location

`/etc/nginx/sites-available/backend`

## Configuration

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

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/cyberrangebackend1.bhumiitech.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/cyberrangebackend1.bhumiitech.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = cyberrangebackend1.bhumiitech.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    server_name cyberrangebackend1.bhumiitech.com;
    listen 80;
    return 404; # managed by Certbot
}
```

## Installation

### Step 1: Create Configuration File

```bash
sudo nano /etc/nginx/sites-available/backend
```

Copy the configuration above into the file.

**Important**: Update the following:
- `server_name`: Your domain name
- `alias` in `/static/` location: Path to your static files directory
- `proxy_pass`: Backend server address (default: `http://127.0.0.1:8000`)

### Step 2: Enable the Site

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### Step 3: Setup SSL Certificate

```bash
sudo certbot --nginx -d cyberrangebackend1.bhumiitech.com
```

Certbot will automatically update the configuration with SSL settings.

## Configuration Details

### Static Files

The `/static/` location block:
- Serves static files directly from the filesystem
- Sets CORS headers for cross-origin requests
- Enables directory listing with `autoindex on`

### Proxy Settings

The main location block:
- Proxies all requests to the Daphne server on port 8000
- Sets proper headers for WebSocket support
- Handles X-Forwarded-For for client IP tracking

### SSL Configuration

SSL settings are managed by Certbot and include:
- SSL certificate and key paths
- SSL protocol options
- Diffie-Hellman parameters

## Management Commands

```bash
# Test configuration
sudo nginx -t

# Reload Nginx (apply changes without downtime)
sudo systemctl reload nginx

# Restart Nginx
sudo systemctl restart nginx

# Check Nginx status
sudo systemctl status nginx

# View error logs
sudo tail -f /var/log/nginx/error.log

# View access logs
sudo tail -f /var/log/nginx/access.log
```

## Troubleshooting

### Configuration Test Fails

```bash
# Test configuration for syntax errors
sudo nginx -t

# View detailed error messages
sudo nginx -T
```

### 502 Bad Gateway

1. Verify Daphne service is running:
   ```bash
   sudo systemctl status daphne.service
   ```

2. Check if port 8000 is listening:
   ```bash
   sudo netstat -tlnp | grep 8000
   ```

3. Verify proxy_pass URL is correct

### Static Files Not Serving

1. Verify static files directory exists:
   ```bash
   ls -la /home/ubuntu/rangestorm2.5/cyber_range_platform_v2/static/
   ```

2. Check permissions:
   ```bash
   sudo chown -R www-data:www-data /home/ubuntu/rangestorm2.5/cyber_range_platform_v2/static/
   ```

3. Collect static files in Django:
   ```bash
   python manage.py collectstatic
   ```

### SSL Certificate Issues

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
   ```

### CORS Issues

If you encounter CORS errors:
1. Verify the `Access-Control-Allow-Origin` header matches your frontend URL
2. Check that the frontend is making requests to the correct domain
3. Ensure CORS settings in Django settings.py are configured

## Related Documentation

- [Backend Deployment Guide](../README.md)
- [Daphne Service Configuration](../services/daphne/README.md)
- [Main Deployment Guide](../../README.md)

---

*For additional help, refer to the [Backend Deployment Guide](../README.md)*

