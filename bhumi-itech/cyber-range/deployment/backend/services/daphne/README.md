# Daphne Service Configuration

Systemd service for Daphne ASGI server.

## Service File
Location: `/etc/systemd/system/daphne.service`

## Management
- Start: `sudo systemctl start daphne.service`
- Status: `sudo systemctl status daphne.service`
- Logs: `journalctl -u daphne.service -f`

For full documentation, see [Backend Deployment Guide](../README.md).