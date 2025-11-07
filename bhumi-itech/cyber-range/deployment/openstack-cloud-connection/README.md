# OpenStack Cloud Connection Configuration

Guide for configuring OpenStack cloud integration.

## Key Configuration

### Update Router Name
In `cloud_management/utils.py`, update:
```python
def connect_router_to_public_network(router, public_network_name="Public-Lan"):
    # Change to your actual public network name
```

### Update Availability Zone
In `cloud_management/utils.py`, update:
```python
def create_cloud_instance(..., instance_availability_zone="Compute1.bhumiitech.com"):
    # Change to your actual availability zone (default: "nova")
```

For full documentation, see [Main Deployment Guide](../README.md).