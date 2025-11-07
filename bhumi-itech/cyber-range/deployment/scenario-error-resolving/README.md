# Additional Web Based Scenario Error Resolving

Guide for resolving errors related to web-based scenario creation and deployment.

## 📋 Table of Contents

- [Overview](#overview)
- [Common Issues](#common-issues)
- [Network Configuration](#network-configuration)
- [Availability Zone Issues](#availability-zone-issues)
- [Troubleshooting Steps](#troubleshooting-steps)

---

## Overview

This guide addresses common errors encountered when creating and deploying web-based scenarios in the Cyber Range platform. Most issues are related to OpenStack configuration, network settings, or availability zone mismatches.

---

## Common Issues

### Issue 1: Scenario Instances Not Being Created

**Symptoms**:
- Scenario creation starts but instances never appear
- Error messages about network connectivity
- Timeout errors during instance creation

**Common Causes**:
- Incorrect public network router name
- Wrong availability zone configuration
- Network connectivity issues
- OpenStack API connection problems

---

## Network Configuration

### Problem: Router Name Mismatch

If scenario instances are not being created, the public network router name may be incorrect.

### Solution

#### Step 1: Locate Configuration File

```bash
cd cloud_management
nano utils.py
```

#### Step 2: Find and Update Router Function

Locate the `connect_router_to_public_network` function:

```python
def connect_router_to_public_network(router, public_network_name="Public-Lan"):
    # Change "Public-Lan" to your actual public network name
```

**Action Required**: Update `public_network_name` parameter to match your OpenStack public network name.

**Example**:
```python
def connect_router_to_public_network(router, public_network_name="external"):
    # Updated to match your OpenStack network name
```

### Finding Your Network Name

1. **Via OpenStack Dashboard**:
   - Log into Horizon Dashboard
   - Navigate to **Network** → **Networks**
   - Find network marked as **External**
   - Note the exact name

2. **Via OpenStack CLI**:
   ```bash
   openstack network list --external
   ```

---

## Availability Zone Issues

### Problem: Wrong Availability Zone

The default availability zone may not exist or may not have resources available.

### Solution

#### Step 1: Locate Configuration File

```bash
cd cloud_management
nano utils.py
```

#### Step 2: Find and Update Instance Creation Function

Locate the `create_cloud_instance` function:

```python
def create_cloud_instance(instance_name, instance_image_id, instance_flavor_id, instance_network_id, instance_availability_zone="Compute1.bhumiitech.com"):
    # Change availability zone to match your OpenStack setup
```

**Action Required**: Update `instance_availability_zone` parameter.

**Options**:
- Use `"nova"` for default zone
- Use specific zone name from your OpenStack setup
- Leave empty to use default

**Example**:
```python
def create_cloud_instance(instance_name, instance_image_id, instance_flavor_id, instance_network_id, instance_availability_zone="nova"):
    # Updated to use default nova zone
```

### Finding Your Availability Zone

1. **Via OpenStack Dashboard**:
   - Navigate to **Compute** → **Instances**
   - Click **Launch Instance**
   - Check **Availability Zone** dropdown
   - Note available zones

2. **Via OpenStack CLI**:
   ```bash
   openstack availability zone list --compute
   ```

---

## Troubleshooting Steps

### Step 1: Verify OpenStack Connection

```bash
# Test OpenStack API connection
openstack server list

# Check network connectivity
openstack network list
```

### Step 2: Check Application Logs

```bash
# Check backend logs
journalctl -u daphne.service -f

# Check Celery worker logs
journalctl -u celery-worker.service -f
```

### Step 3: Verify Configuration

1. Check `.env` file for OpenStack credentials:
   ```env
   AUTH_URL = 'http://10.1.75.40:5000'
   PROJECT_ID = 'your-project-id'
   USERNAME = 'your-username'
   PASSWORD = 'your-password'
   ```

2. Verify network and zone names match OpenStack

### Step 4: Test Instance Creation Manually

```bash
# Try creating an instance via OpenStack CLI
openstack server create \
  --image <image-id> \
  --flavor <flavor-id> \
  --network <network-id> \
  --availability-zone nova \
  test-instance
```

---

## Complete Configuration Example

Here's a complete example of properly configured functions:

```python
# cloud_management/utils.py

def connect_router_to_public_network(router, public_network_name="external"):
    """
    Connect router to public network.
    
    IMPORTANT: Update public_network_name to match your OpenStack setup.
    Common names: "external", "public", "Public-Lan", "floating"
    """
    # Implementation here
    pass

def create_cloud_instance(instance_name, instance_image_id, instance_flavor_id, instance_network_id, instance_availability_zone="nova"):
    """
    Create a cloud instance.
    
    IMPORTANT: Update instance_availability_zone to match your OpenStack setup.
    Common zones: "nova" (default), "Compute1", "zone-1"
    """
    # Implementation here
    pass
```

---

## Verification Checklist

After making changes, verify:

- [ ] Network name matches OpenStack dashboard
- [ ] Availability zone exists and has resources
- [ ] OpenStack credentials are correct in `.env`
- [ ] Router is connected to public network
- [ ] Test instance creation works
- [ ] Application logs show no errors

---

## Additional Resources

- [OpenStack Cloud Connection Guide](../openstack-cloud-connection/README.md)
- [Backend Deployment Guide](../backend/README.md)
- [Main Deployment Guide](../README.md)

---

## Quick Reference

### Common Network Names
- `external`
- `public`
- `Public-Lan`
- `floating`
- `ext-net`

### Common Availability Zones
- `nova` (default)
- `Compute1`
- `zone-1`
- `az1`

---

*For additional help, refer to the [OpenStack Cloud Connection Guide](../openstack-cloud-connection/README.md)*

