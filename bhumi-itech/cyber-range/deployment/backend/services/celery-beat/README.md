# Celery Beat Service Configuration

Systemd service for Celery Beat scheduler.

## Service File
Location: `/etc/systemd/system/celery-beat.service`

## Management
- Start: `sudo systemctl start celery-beat.service`
- Status: `sudo systemctl status celery-beat.service`
- Logs: `journalctl -u celery-beat.service -f`

For full documentation, see [Backend Deployment Guide](../README.md).