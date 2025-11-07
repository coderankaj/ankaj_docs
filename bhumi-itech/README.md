# Bhumi Itech - Cyber Range Platform

Documentation for the Cyber Range platform deployment and management.

## 📋 Overview

The Cyber Range platform is a comprehensive cybersecurity training and simulation environment built with Django backend, React frontend, and MongoDB database.

## 🏗️ Architecture

- **Backend**: Django with Daphne (ASGI server)
- **Frontend**: React.js
- **Database**: MongoDB
- **Task Queue**: Celery with Redis
- **Web Server**: Nginx
- **Process Manager**: PM2 (for dashboard)

## 📚 Documentation

### [Deployment Guide](cyber-range/deployment/README.md)
Complete step-by-step deployment instructions for all components.

### Components

- [Backend Services](cyber-range/deployment/backend/README.md) - Django, Celery, Daphne
- [MongoDB Database](cyber-range/deployment/mongodb/README.md) - Database setup and configuration
- [Frontend Application](cyber-range/deployment/README.md#frontend-deployment) - React.js deployment
- [Super User Dashboard](cyber-range/deployment/README.md#super-user-dashboard) - Next.js dashboard
- [OpenStack Integration](cyber-range/deployment/README.md#openstack-configuration) - Cloud integration

## 🔧 Service Management

### Backend Services

```bash
# Daphne (ASGI Server)
sudo systemctl start daphne.service
sudo systemctl restart daphne.service
sudo systemctl status daphne.service
journalctl -u daphne.service -f

# Celery Worker
sudo systemctl start celery-worker
sudo systemctl restart celery-worker
sudo systemctl status celery-worker
journalctl -u celery-worker -f

# Celery Beat (Scheduler)
sudo systemctl enable celery-beat
sudo systemctl start celery-beat
sudo systemctl restart celery-beat
sudo systemctl status celery-beat
journalctl -u celery-beat -f
```

## 🔐 Credentials

⚠️ **Security Note**: Credentials are stored separately. See [credentials](credentials) file for reference.

## 📖 Quick Links

- [Full Deployment Guide](cyber-range/deployment/README.md)
- [Troubleshooting](cyber-range/deployment/README.md#troubleshooting)

---

*For detailed deployment instructions, see the [Deployment Guide](cyber-range/deployment/README.md)*

