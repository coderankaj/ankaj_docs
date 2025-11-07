# Frontend React Deployment Guide

Complete guide for deploying the React.js frontend application.

## Prerequisites
- Node.js and npm installed
- Build tools available
- Nginx configured
- Access to server at `/home/ubuntu/rangestorm2.5/cyber_range_frontend_v2.5`

## Build Commands

### Development Build
```bash
GENERATE_SOURCEMAP=false NODE_OPTIONS="--max-old-space-size=8192" npm run build:dev
```

### Production Build
For 4GB memory:
```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run build:prod
```

For 8GB memory:
```bash
NODE_OPTIONS="--max-old-space-size=8192" npm run build:prod
```

## Deployment Steps

1. Navigate to project: `cd /home/ubuntu/rangestorm2.5/cyber_range_frontend_v2.5`
2. Build: `npm run build:dev` or `npm run build:prod`
3. Deploy:
```bash
sudo mv /var/www/html /var/www/html.bak-$(date +%Y%m%d-%H%M%S)
sudo cp -r ~/rangestorm2.5/cyber_range_frontend_v2.5/build/. /var/www/html
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
sudo nginx -t
sudo systemctl restart nginx
```

For full documentation, see [Main Deployment Guide](../README.md).