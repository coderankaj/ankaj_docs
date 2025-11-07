# MongoDB Setup Guide

Complete guide for installing and configuring MongoDB for the Cyber Range platform.

## 📋 Table of Contents

- [Installation](#installation)
- [Security Configuration](#security-configuration)
- [User Management](#user-management)
- [Connection Configuration](#connection-configuration)
- [Troubleshooting](#troubleshooting)

---

## Installation

### Step 1: Install Prerequisites

```bash
sudo apt-get install gnupg curl
```

### Step 2: Add MongoDB GPG Key

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```

### Step 3: Add MongoDB Repository

```bash
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/8.2 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.2.list
```

### Step 4: Install MongoDB

```bash
# Update package list
sudo apt-get update

# Install MongoDB
sudo apt-get install -y mongodb-org
```

### Step 5: Start MongoDB

```bash
# Start MongoDB service
sudo systemctl start mongod

# Enable MongoDB to start on boot
sudo systemctl enable mongod

# Verify MongoDB is running
sudo systemctl status mongod
```

---

## Security Configuration

### Step 1: Edit MongoDB Configuration

```bash
sudo nano /etc/mongod.conf
```

### Step 2: Update Configuration

Update the configuration file with the following settings:

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0  # Listen on all interfaces (change to specific IP for production)

security:
  authorization: enabled  # Enable authentication
```

**Security Note**: For production environments, consider binding to a specific IP address instead of `0.0.0.0`.

### Step 3: Restart MongoDB

```bash
sudo systemctl restart mongod
```

---

## User Management

### Step 1: Connect to MongoDB

```bash
mongosh
```

### Step 2: Create Database User

Create a user for the `cyber_range` database:

```javascript
use cyber_range

db.createUser({
  user: "range",
  pwd: "idrbtprorange",
  roles: [ { role: "readWrite", db: "cyber_range" } ]
})
```

### Step 3: Create Admin User

Create an admin user with root privileges:

```javascript
use admin

db.createUser({
  user: "admin",
  pwd: "Cyberrange@365&2829",
  roles: [ { role: "root", db: "admin" } ]
})
```

### Step 4: Verify Users

```javascript
// Switch to admin database
use admin

// Authenticate as admin
db.auth("admin", "Cyberrange@365&2829")

// List all users
db.getUsers()

// Check cyber_range database users
use cyber_range
db.getUsers()
```

---

## Connection Configuration

### Connection String Format

```
mongodb://username:password@host:port/
```

### Example Connection Strings

#### For Application Use

```env
MONGO_DB_URI = "mongodb://range:idrbtprorange@10.1.20.75:27017/"
```

#### For Celery

```env
HOST = "mongodb://range:idrbtprorange@10.1.20.75:27017/"
DATABASE = "cyber_range"
TASKMETA_COLLECTION = "celery_taskmeta"
```

### Testing Connection

#### Using mongosh

```bash
# Connect with authentication
mongosh "mongodb://range:idrbtprorange@10.1.20.75:27017/cyber_range"

# Or connect and authenticate separately
mongosh
use cyber_range
db.auth("range", "idrbtprorange")
```

#### Using Python

```python
from pymongo import MongoClient

# Connect to MongoDB
client = MongoClient("mongodb://range:idrbtprorange@10.1.20.75:27017/")

# Access database
db = client.cyber_range

# Test connection
print(db.list_collection_names())
```

---

## Database Management

### Common Operations

#### List Databases

```javascript
show dbs
```

#### Switch Database

```javascript
use cyber_range
```

#### List Collections

```javascript
show collections
```

#### Create Collection

```javascript
db.createCollection("collection_name")
```

#### Drop Collection

```javascript
db.collection_name.drop()
```

#### Backup Database

```bash
mongodump --uri="mongodb://range:idrbtprorange@10.1.20.75:27017/cyber_range" --out=/backup/path
```

#### Restore Database

```bash
mongorestore --uri="mongodb://range:idrbtprorange@10.1.20.75:27017/cyber_range" /backup/path/cyber_range
```

---

## Troubleshooting

### MongoDB Not Starting

1. Check service status:
   ```bash
   sudo systemctl status mongod
   ```

2. View MongoDB logs:
   ```bash
   sudo journalctl -u mongod -f
   ```

3. Check configuration file syntax:
   ```bash
   sudo mongod --config /etc/mongod.conf --test
   ```

### Connection Refused

1. Verify MongoDB is running:
   ```bash
   sudo systemctl status mongod
   ```

2. Check if MongoDB is listening on the correct port:
   ```bash
   sudo netstat -tlnp | grep 27017
   ```

3. Verify firewall settings:
   ```bash
   sudo ufw status
   ```

### Authentication Failed

1. Verify user exists:
   ```javascript
   use admin
   db.auth("admin", "Cyberrange@365&2829")
   db.getUsers()
   ```

2. Check if authorization is enabled:
   ```bash
   grep -i authorization /etc/mongod.conf
   ```

3. Verify credentials in connection string

### Permission Denied

1. Verify user has correct roles:
   ```javascript
   use cyber_range
   db.getUser("range")
   ```

2. Grant additional permissions if needed:
   ```javascript
   db.grantRolesToUser("range", [{ role: "readWrite", db: "cyber_range" }])
   ```

### Port Already in Use

If port 27017 is already in use:

```bash
# Find process using port 27017
sudo lsof -i :27017

# Or check if another MongoDB instance is running
ps aux | grep mongod
```

### Disk Space Issues

Check disk space:

```bash
df -h
```

Clean up MongoDB logs if needed:

```bash
# Rotate logs
sudo systemctl restart mongod
```

---

## Security Best Practices

1. **Change Default Passwords**: Always use strong, unique passwords
2. **Network Binding**: In production, bind to specific IP addresses
3. **Firewall**: Restrict MongoDB port access to trusted IPs
4. **Regular Backups**: Schedule regular database backups
5. **Monitor Logs**: Regularly check MongoDB logs for suspicious activity
6. **Update Regularly**: Keep MongoDB updated to the latest stable version

---

## Related Documentation

- [Backend Deployment](../backend/README.md)
- [Main Deployment Guide](../README.md)

---

*For additional help, refer to the [Backend Deployment Guide](../backend/README.md)*

