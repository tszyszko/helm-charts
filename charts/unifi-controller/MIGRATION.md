# Migration Guide: UniFi Controller to UniFi Network Application

This guide covers migrating from chart version 2.x (using `linuxserver/unifi-controller`) to version 3.x (using `linuxserver/unifi-network-application` with MongoDB 8).

## Overview

The chart has been completely redesigned to use the new `linuxserver/unifi-network-application` image, which requires an external MongoDB database. This is a **breaking change** that requires manual migration.

## Pre-Migration Steps

### 1. Backup Your Configuration

Before starting the migration, create a complete backup of your UniFi configuration:

1. Access your UniFi Controller web interface (https://your-controller:8443)
2. Go to Settings → System → Backup
3. Create a backup and download it to a safe location
4. **Important**: Include history in your backup if you want to preserve statistics

### 2. Document Current Settings

Note down your current configuration:
- Network settings
- Device configurations
- User accounts
- Application settings

## Migration Process

### Step 1: Prepare New Values

Create a new `values.yaml` file with MongoDB configuration:

```yaml
# MongoDB configuration (required)
mongodb:
  enabled: true
  auth:
    enabled: true
    rootUsername: root
    rootPassword: "your-secure-root-password"    # Set a strong password
    username: unifi
    password: "your-secure-unifi-password"       # Set a strong password
    database: unifi
  persistence:
    enabled: true
    size: 8Gi
    storageClass: ""  # Use your preferred storage class

# UniFi Network Application environment
environment:
  timezone: "Europe/Berlin"  # Set your timezone
  mongoPass: "your-secure-unifi-password"  # Must match mongodb.auth.password

# Service configuration
service:
  type: "LoadBalancer"
  loadBalancerIP: ""  # Set your current IP if using a specific one

# Ingress configuration (update your domain)
ingress:
  enabled: true
  hosts:
    - host: unifi.your-domain.com
      paths:
        - path: /
          pathType: ImplementationSpecific
```

### Step 2: Uninstall Old Chart

```bash
# Scale down the old deployment to avoid conflicts
kubectl scale deployment unifi-controller --replicas=0

# Uninstall the old chart (but keep PVCs)
helm uninstall unifi-controller

# Verify old resources are cleaned up
kubectl get pods,services -l app.kubernetes.io/name=unifi-controller
```

### Step 3: Install New Chart

```bash
# Install the new chart version
helm install unifi-controller ./charts/unifi-controller \
  -f your-new-values.yaml \
  --version 3.0.0
```

### Step 4: Restore Configuration

1. Wait for the new pods to be ready:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=unifi-controller
   ```

2. Access the new UniFi Network Application interface
3. Complete the initial setup wizard
4. **Important**: Choose "Restore from backup" during setup
5. Upload your backup file from Step 1
6. Follow the restoration process

### Step 5: Verify Migration

1. Check that all devices are reconnected
2. Verify network settings are correct
3. Test device adoption if needed
4. Confirm statistics/history are preserved (if included in backup)

## Post-Migration Tasks

### Update Device Inform URLs

If devices aren't connecting automatically, you may need to update the inform URL:

1. SSH into each device: `ssh ubnt@device-ip`
2. Update inform URL: `set-inform http://your-new-controller:8080/inform`

### Update DNS/Firewall Rules

- Update any DNS entries pointing to the old controller
- Update firewall rules if the IP address changed
- Update any monitoring systems

## Troubleshooting

### MongoDB Connection Issues

If UniFi can't connect to MongoDB:

1. Check MongoDB pod logs:
   ```bash
   kubectl logs -l app.kubernetes.io/component=mongodb
   ```

2. Verify MongoDB credentials match between `mongodb.auth.password` and `environment.mongoPass`

3. Check if MongoDB initialization completed:
   ```bash
   kubectl exec -it deployment/unifi-controller-mongodb -- mongosh
   ```

### Device Adoption Problems

If devices won't adopt:

1. Check the inform host setting in Settings → System → Advanced
2. Ensure port 8080 is accessible from your network
3. Manual adoption via SSH (see above)

### Performance Issues

If the new setup is slower:

1. Increase MongoDB resources in `values.yaml`:
   ```yaml
   mongodb:
     resources:
       requests:
         memory: 1Gi
         cpu: 500m
   ```

2. Adjust UniFi memory settings:
   ```yaml
   environment:
     memLimit: 2048
     memStartup: 1024
   ```

## Rollback Plan

If migration fails, you can rollback:

1. Keep your backup files safe
2. Reinstall the old chart version (2.x)
3. Restore from your backup

```bash
# Emergency rollback
helm install unifi-controller ./charts/unifi-controller \
  --version 2.6.1 \
  -f your-old-values.yaml
```

## Support

- Check the [README.md](./README.md) for configuration options
- Review the [values-example.yaml](./values-example.yaml) for a complete example
- See MongoDB 8 documentation for database-specific issues
- Check LinuxServer.io documentation for image-specific problems 