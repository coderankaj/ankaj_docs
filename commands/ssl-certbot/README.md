# SSL Certificate Setup with Let's Encrypt (Certbot)

Complete guide for installing and managing Let's Encrypt SSL certificates on Ubuntu servers using Certbot.

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Generating Certificates](#generating-certificates)
- [Certificate Management](#certificate-management)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Overview

Let's Encrypt provides free SSL/TLS certificates that are trusted by all major browsers. Certbot is the official client tool for obtaining and managing these certificates. This guide covers installation and usage on Ubuntu servers, including AWS EC2 instances.

**Reference**: [How to install Letsencrypt SSL certificate on AWS EC2 ubuntu instance](https://www.webcreta.com/how-to-letsencrypt-ssl-certificate-install-on-aws-ec2-ubuntu-instance/)

---

## Prerequisites

- Ubuntu server (including AWS EC2 instances)
- Root or sudo access
- Domain name pointing to your server's IP address
- Web server installed (Apache or Nginx)
- Ports 80 and 443 open in firewall/security groups

### AWS EC2 Specific Requirements

- Security Group configured to allow:
  - Inbound HTTP (port 80) from `0.0.0.0/0`
  - Inbound HTTPS (port 443) from `0.0.0.0/0`
- Domain DNS A record pointing to EC2 instance's public IP

---

## Installation

### Step 1: Update System Packages

```bash
sudo apt-get update
```

### Step 2: Install Software Properties Common

```bash
sudo apt-get install software-properties-common
```

### Step 3: Add Certbot Repository

```bash
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

### Step 4: Install Certbot

#### For Apache Web Server

```bash
sudo apt-get install python-certbot-apache
```

#### For Nginx Web Server

```bash
sudo apt-get install python-certbot-nginx
```

#### For Standalone Installation (No Web Server Plugin)

```bash
sudo apt-get install certbot
```

---

## Generating Certificates

### Single Domain Certificate

#### For Apache

```bash
sudo certbot --apache -d yourdomain.com
```

#### For Nginx

```bash
sudo certbot --nginx -d yourdomain.com
```

### Multiple Domains/Subdomains Certificate

You can generate a certificate for multiple domains or subdomains in a single command. The first domain in the list is the base domain.

#### For Apache

```bash
sudo certbot --apache -d yourdomain.com -d www.yourdomain.com
```

#### For Nginx

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

### Wildcard Certificate (DNS Challenge)

For wildcard certificates, you need to use DNS validation:

```bash
sudo certbot certonly --manual --preferred-challenges dns -d *.yourdomain.com -d yourdomain.com
```

Follow the prompts to add DNS TXT records for verification.

### Standalone Mode (No Web Server Plugin)

If you don't want Certbot to modify your web server configuration:

```bash
sudo certbot certonly --standalone -d yourdomain.com -d www.yourdomain.com
```

**Note**: This requires stopping your web server temporarily during certificate generation.

---

## Certificate Storage

Certificates are stored in the following location:

```
/etc/letsencrypt/live/yourdomain.com/
```

### Certificate Files

- `fullchain.pem` - Full certificate chain (certificate + intermediate certificates)
- `privkey.pem` - Private key
- `cert.pem` - Certificate only
- `chain.pem` - Intermediate certificates only

### Configuration Files

Certbot automatically updates your web server configuration files:

- **Apache**: `/etc/apache2/sites-available/yourdomain.com.conf`
- **Nginx**: `/etc/nginx/sites-available/yourdomain.com`

---

## Certificate Management

### Check Certificate Status

```bash
sudo certbot certificates
```

### Test Certificate Renewal (Dry Run)

```bash
sudo certbot renew --dry-run
```

### Renew Certificates Manually

```bash
sudo certbot renew
```

### Auto-Renewal Setup

Certbot automatically sets up a systemd timer or cron job for renewal. Verify it's active:

```bash
# Check systemd timer
sudo systemctl status certbot.timer

# Enable if not already enabled
sudo systemctl enable certbot.timer
```

### Renew Specific Certificate

```bash
sudo certbot renew --cert-name yourdomain.com
```

### Revoke Certificate

If you need to revoke a certificate:

```bash
sudo certbot revoke --cert-path /etc/letsencrypt/live/yourdomain.com/cert.pem
```

### Delete Certificate

To completely remove a certificate:

```bash
sudo certbot delete --cert-name yourdomain.com
```

---

## Web Server Configuration

### Apache Configuration

After running Certbot with `--apache`, your configuration will be automatically updated. Example:

```apache
<VirtualHost *:443>
    ServerName yourdomain.com
    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/yourdomain.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/yourdomain.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateChainFile /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
```

### Nginx Configuration

After running Certbot with `--nginx`, your configuration will be automatically updated. Example:

```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Your other configuration...
}

server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}
```

---

## Troubleshooting

### Certificate Generation Fails

**Problem**: Cannot connect to domain for validation

**Solutions**:
1. Verify DNS A record points to your server IP
2. Check firewall/security group allows port 80
3. Ensure web server is running and accessible
4. Test with: `curl http://yourdomain.com`

### Port 80 Already in Use

**Problem**: Certbot cannot bind to port 80

**Solution**: Stop the service using port 80 temporarily:
```bash
sudo systemctl stop apache2  # or nginx
sudo certbot certonly --standalone -d yourdomain.com
sudo systemctl start apache2  # or nginx
```

### Certificate Renewal Fails

**Problem**: Auto-renewal not working

**Solutions**:
1. Check Certbot logs: `sudo tail -f /var/log/letsencrypt/letsencrypt.log`
2. Test renewal manually: `sudo certbot renew --dry-run`
3. Verify systemd timer: `sudo systemctl status certbot.timer`

### SSL Certificate Expired

**Problem**: Certificate has expired

**Solution**: Renew immediately:
```bash
sudo certbot renew --force-renewal
sudo systemctl reload apache2  # or nginx
```

### Multiple Certificates for Same Domain

**Problem**: Multiple certificates exist

**Solution**: List and delete unwanted certificates:
```bash
sudo certbot certificates
sudo certbot delete --cert-name unwanted-cert-name
```

### AWS EC2 Specific Issues

**Problem**: Cannot validate domain on EC2

**Solutions**:
1. Check Security Group inbound rules for ports 80 and 443
2. Verify Elastic IP is assigned (if using)
3. Ensure domain DNS propagation is complete
4. Check EC2 instance public IP matches DNS A record

---

## Best Practices

1. **Auto-Renewal**: Always enable auto-renewal to prevent certificate expiration
2. **Backup**: Regularly backup `/etc/letsencrypt/` directory
3. **Monitoring**: Set up monitoring for certificate expiration (90 days before expiry)
4. **Multiple Domains**: Use single certificate for related domains to simplify management
5. **Wildcard Certificates**: Use for subdomains when possible
6. **Test Renewal**: Regularly test renewal with `--dry-run` flag

---

## Common Commands Reference

```bash
# Install Certbot for Nginx
sudo apt-get install python-certbot-nginx

# Generate certificate for single domain
sudo certbot --nginx -d yourdomain.com

# Generate certificate for multiple domains
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Check certificate status
sudo certbot certificates

# Test renewal
sudo certbot renew --dry-run

# Renew all certificates
sudo certbot renew

# Renew specific certificate
sudo certbot renew --cert-name yourdomain.com

# Revoke certificate
sudo certbot revoke --cert-path /etc/letsencrypt/live/yourdomain.com/cert.pem

# Delete certificate
sudo certbot delete --cert-name yourdomain.com
```

---

## References

- [Official Certbot Documentation](https://certbot.eff.org/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [How to install Letsencrypt SSL certificate on AWS EC2 ubuntu instance](https://www.webcreta.com/how-to-letsencrypt-ssl-certificate-install-on-aws-ec2-ubuntu-instance/)

---

## Related Documentation

- [Commands Reference](../README.md)
- [Backend Nginx Configuration](../../bhumi-itech/cyber-range/deployment/backend/nginx/README.md)
- [Super User Dashboard SSL Setup](../../bhumi-itech/cyber-range/deployment/super-user-dashboard/README.md#ssl-setup)

---

*Last updated: 2024*

