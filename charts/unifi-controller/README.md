# UniFi Network Application Helm Chart

The UniFi Network Application helm chart installs a UniFi Network Application (formerly UniFi Controller) with MongoDB 8 onto your Kubernetes cluster. A Kubernetes cluster (k3s, kind, K8s, or anything else) is required. For exposing the ports, MetalLB or any other LoadBalancer implementation is required.

## ⚠️ Breaking Changes in v3.0.0

**This chart has been migrated from the deprecated `linuxserver/unifi-controller` to the new `linuxserver/unifi-network-application` image.**

### Key Changes:
- **External MongoDB Required**: The new image requires an external MongoDB database (included in this chart)
- **MongoDB 8.0**: Uses MongoDB 8.0 for optimal compatibility
- **New Environment Variables**: Updated configuration for the new image
- **Image Repository**: Changed from `linuxserver/unifi-controller` to `linuxserver/unifi-network-application`

### Migration from v2.x

1. **Backup your existing UniFi configuration** via the web UI (Settings > System > Backup)
2. **Update your values.yaml** to include MongoDB configuration:
   ```yaml
   mongodb:
     enabled: true
     auth:
       rootPassword: "your-secure-root-password"
       password: "your-secure-unifi-password"
   
   environment:
     mongoPass: "your-secure-unifi-password"  # Must match mongodb.auth.password
   ```
3. **Upgrade the chart** to v3.0.0
4. **Restore your configuration** using the backup from step 1

## Configuration

### MongoDB Configuration

The chart deploys MongoDB 8.0 by default. You can configure it using the `mongodb` section in values.yaml:

```yaml
mongodb:
  enabled: true
  auth:
    enabled: true
    rootPassword: "secure-root-password"
    password: "secure-unifi-password"
  persistence:
    enabled: true
    size: 8Gi
```

### External MongoDB

To use an external MongoDB instance, set `mongodb.enabled: false` and configure the connection:

```yaml
mongodb:
  enabled: false

environment:
  mongoUser: unifi
  mongoPass: "your-password"
  mongoHost: "your-mongodb-host"
  mongoPort: 27017
  mongoDbName: unifi
  mongoAuthSource: admin
```

## Controller ports

The following is a list of ports that are used by the UniFi Network Application. The ports are exposed by a MetalLB service to be discoverable in your network.

|Protocol|Port|Purpose|
|--|--|--|
|UDP|3478|Port used for STUN.|
|UDP|5514|Port used for remote syslog capture|
|TCP|8080|Port used for device and application communication|
|TCP|8443|Port used for application GUI/API as seen in a web browser|
|TCP|8880|Port used for HTTP portal redirection|
|TCP|8843|Port used for HTTPS portal redirection|
|TCP|6789|Port used for UniFi mobile speed test|
|UDP|5656-5699|Ports used by AP-EDU broadcasting|
|UDP|10001|Port used for device discovery|
|UDP|1900|Port used for "Make application discoverable on L2 network" in the UniFi Network settings|

## Further reading

- The original chart from Lukas Bahr: [https://github.com/lukibahr/unifi-controller-helm-chart](https://github.com/lukibahr/unifi-controller-helm-chart)
- Releaser action used for releasing the charts: [https://github.com/helm/chart-releaser-action](https://github.com/helm/chart-releaser-action)
- The new image from linuxserver.io: [https://hub.docker.com/r/linuxserver/unifi-network-application](https://hub.docker.com/r/linuxserver/unifi-network-application)
- Official documentation: [https://docs.linuxserver.io/images/docker-unifi-network-application](https://docs.linuxserver.io/images/docker-unifi-network-application)
- UniFi Network Application ports: [https://help.ui.com/hc/en-us/articles/218506997-UniFi-Ports-Used](https://help.ui.com/hc/en-us/articles/218506997-UniFi-Ports-Used)
