# Super User Dashboard Deployment Guide

Complete guide for deploying the CyberRange Super User Dashboard using Next.js, PM2, and Nginx.

## 📖 Complete Documentation

For comprehensive deployment guide covering all aspects of Next.js deployment with PM2 and Nginx, see:

**[Next.js Application Deployment with PM2 and Nginx](../nextjs-pm2-nginx-deployment/README.md)**

This global guide includes:
- Complete step-by-step instructions
- PM2 management and advanced features
- Nginx configuration details
- SSL setup with Let's Encrypt
- Troubleshooting guide
- Best practices

## Quick Start

1. Clone: `git clone https://github.com/coderankaj/cyberrange_dashboard_nextjs.git`
2. Install: `npm i && npm run build`
3. Start with PM2: `pm2 start npm --name cyberrange_dashboard_nextjs -- start`
4. Configure Nginx reverse proxy
5. Setup SSL: `sudo certbot --nginx -d dashboard.bhumiitech.com`

## PM2 Management
- Status: `pm2 status`
- Restart: `pm2 restart cyberrange_dashboard_nextjs`
- Logs: `pm2 logs cyberrange_dashboard_nextjs`

## Related Documentation

- [Complete Next.js Deployment Guide](../nextjs-pm2-nginx-deployment/README.md)
- [SSL Certificate Setup](../../../../commands/ssl-certbot/README.md)
- [Main Deployment Guide](../README.md)