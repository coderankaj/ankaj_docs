# Celery Worker Service Configuration

Systemd service for Celery worker.

## Service File
Location: `/etc/systemd/system/celery-worker.service`

## Management
- Start: `sudo systemctl start celery-worker.service`
- Status: `sudo systemctl status celery-worker.service`
- Logs: `journalctl -u celery-worker.service -f`

For full documentation, see [Backend Deployment Guide](../README.md).