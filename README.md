# openclaw-helm-chart
A community made OpenClaw Helm chart for installing on Kubernetes with Helm

## ⚠️ Experimental & Early Stage

**This chart is experimental and early.** OpenClaw is a stateful, single-instance application and does NOT follow traditional Kubernetes philosophy. This chart wraps OpenClaw for convenience, but:

- ❌ **No horizontal scaling**: OpenClaw cannot run multiple replicas (it's not designed for distributed deployments)
- ❌ **Single instance only**: Only 1 pod should run at a time
- ❌ **Stateful by design**: OpenClaw manages its own state and persists data to disk
- ⚠️ **Early integration**: This is a community chart. OpenClaw may make breaking changes that affect Kubernetes deployments

This chart is suitable for:
- ✅ Development and testing environments
- ✅ Small production deployments (single instance)
- ✅ Users who want to run OpenClaw on Kubernetes infrastructure

As OpenClaw matures and moves towards distributed architectures, this chart will be updated accordingly.

## About

This Helm chart provides a deployment of [OpenClaw](https://github.com/moltbot/openclaw) on Kubernetes. It includes:

- **Persistent Storage**: Separate PVCs for configuration (`/home/node/.openclaw`) and extraction memory (`/home/node/claw`)
- **Configuration Management**: Init container that sets up config on first start, then uses persisted config on subsequent starts
- **Security Hardened**: Non-root user, dropped capabilities, security policies
- **Networking**: Built-in Service and optional Ingress support with WebSocket configuration examples
- **Hot-Reload Support**: Modify config and restart gateway without pod restart

## Quick Start

### Prerequisites
- Kubernetes 1.19+
- Helm 3.0+

### Installation

Add the Helm repository (when available):
```bash
helm repo add openclaw https://charts.openclaw.io
helm repo update
```

Or install directly from source:
```bash
helm install openclaw ./openclaw-chart
```

### Basic Deployment with Custom Image

```bash
helm install openclaw ./openclaw-chart \
  --set image.repository=your-registry/openclaw \
  --set image.tag=v1.0.0
```

### Enable Ingress

```bash
helm install openclaw ./openclaw-chart \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=openclaw.example.com \
  --set ingress.className=nginx
```

### Set Gateway Token (Authentication)

OpenClaw requires a gateway token for authentication. Set it during deployment:

```bash
helm install openclaw ./openclaw-chart \
  --set env[0].name=OPENCLAW_GATEWAY_TOKEN \
  --set env[0].value="your-secure-token-here"
```

Then access the dashboard with the token in the URL:
```
http://localhost:18789/?token=your-secure-token-here
```

Or visit `http://localhost:18789/` and paste the token in the Control UI settings.

### Initial Setup (Configure Channels)

After deployment, wait for the pod to be READY (5-7 minutes), then:

```bash
# Get pod name
POD=$(kubectl get pod -l app.kubernetes.io/name=openclaw -o jsonpath='{.items[0].metadata.name}')

# Enter the pod shell
kubectl exec -it $POD -- bash

# Run the onboarding wizard (openclaw is in PATH)
openclaw onboard

# Or if PATH is not set:
/openclaw-build/openclaw.mjs onboard
```

The onboarding wizard will walk you through:
- Setting up your AI model (Claude, GPT, etc.)
- Configuring channels (WhatsApp, Telegram, Slack, Discord, etc.)
- Setting up authentication tokens
- Configuring other integrations

After onboarding, restart the pod:
```bash
kubectl delete pod $POD
```

The configuration will persist in the PVC and be loaded on next startup.

### Hot-Reload Configuration Changes

You can modify the gateway configuration without restarting the entire pod:

```bash
# Get pod name
POD=$(kubectl get pod -l app.kubernetes.io/name=openclaw -o jsonpath='{.items[0].metadata.name}')

# Edit the configuration file
kubectl exec -it $POD -- vi /home/node/.openclaw/openclaw.json

# Restart just the gateway (hot-reload)
kubectl exec $POD -- openclaw gateway restart
```

**How it works:**
- **First start**: Gateway checks if config doesn't exist → initializes with `--port 18789 --bind lan --allow-unconfigured` flags
- **Subsequent starts**: Config already exists → loads from persistent storage (CLI args ignored)
- **Hot-reload**: Use `openclaw gateway restart` to reload config without pod restart

This allows you to make configuration changes and immediately apply them without downtime.

### Provide Custom Configuration

```bash
helm install openclaw ./openclaw-chart \
  --set openclawConfig.gateway.port=18789 \
  --set openclawConfig.features.enableBrowser=true
```

Or using a values file:

```yaml
# values.yaml
image:
  repository: openclaw
  tag: latest

env:
  - name: OPENCLAW_GATEWAY_TOKEN
    value: "your-secure-token-here"

openclawConfig:
  gateway:
    port: 18789
  features:
    enableBrowser: true
  database:
    url: "postgres://user:pass@postgres:5432/openclaw"

persistence:
  enabled: true
  config:
    size: 2Gi
  memory:
    size: 10Gi
```

Then install:
```bash
helm install openclaw ./openclaw-chart -f values.yaml
```

## Configuration

### Image Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Image repository | `openclaw` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Pull policy | `IfNotPresent` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type (ClusterIP, NodePort, LoadBalancer) | `ClusterIP` |
| `service.port` | Service port | `18789` |

### Persistence

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent volumes | `true` |
| `persistence.config.size` | Config volume size | `1Gi` |
| `persistence.config.accessMode` | Config access mode | `ReadWriteOnce` |
| `persistence.memory.size` | Memory/extraction volume size | `5Gi` |
| `persistence.memory.accessMode` | Memory access mode | `ReadWriteOnce` |
| `persistence.existingClaim.config` | Use existing PVC for config | `` |
| `persistence.existingClaim.memory` | Use existing PVC for memory | `` |

### OpenClaw Configuration

The `openclawConfig` value is stored in a ConfigMap and merged with persistent configuration during pod initialization:

```yaml
openclawConfig:
  gateway:
    port: 18789
  features:
    enableBrowser: true
    # Add any OpenClaw configuration here
```

**How it works**: On pod startup, the init container (`config-init`) checks if a config exists in the persistent volume. If not, it copies the ConfigMap config. If one exists, it performs a **deep merge** where the ConfigMap values override/patch the persistent config. This allows Helm to update configuration while preserving other persistent values.

### Security Context

| Parameter | Description | Default |
|-----------|-------------|---------|
| `securityContext.runAsNonRoot` | Run as non-root | `true` |
| `securityContext.runAsUser` | User ID | `1000` |
| `securityContext.runAsGroup` | Group ID | `1000` |
| `securityContext.fsGroup` | File system group | `1000` |
| `securityContext.capabilities.drop` | Drop capabilities | `["ALL"]` |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.requests.memory` | Memory request | `512Mi` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.limits.memory` | Memory limit | `2Gi` |
| `resources.limits.cpu` | CPU limit | `none` |

### Ingress

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `nginx` |
| `ingress.hosts[0].host` | Hostname | `openclaw.example.com` |
| `ingress.hosts[0].paths[0].path` | Path | `/` |

**WebSocket Support**: For nginx ingress with WebSocket support, uncomment annotations in values:
```yaml
ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/websocket-services: openclaw
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
```

## Advanced Usage

### Using Existing PVCs

```bash
helm install openclaw ./openclaw-chart \
  --set persistence.existingClaim.config=my-config-pvc \
  --set persistence.existingClaim.memory=my-memory-pvc
```

### Environment Variables

```bash
helm install openclaw ./openclaw-chart \
  --set env[0].name=NODE_ENV \
  --set env[0].value=production \
  --set env[1].name=LOG_LEVEL \
  --set env[1].value=debug
```

### Using Secrets

```yaml
secrets:
  existingSecret:
    name: openclaw-secrets
    keys:
      - api-token
      - api-key
```

### Autoscaling

```bash
helm install openclaw ./openclaw-chart \
  --set autoscaling.enabled=true \
  --set autoscaling.minReplicas=2 \
  --set autoscaling.maxReplicas=10 \
  --set autoscaling.targetCPUUtilizationPercentage=80
```

## Init Container (Config Merge)

The chart includes an init container that performs intelligent config merging:

1. **First deployment**: Copies ConfigMap config to PVC
2. **Subsequent deployments**: Deep merges ConfigMap over existing PVC config

This allows you to:
- Update config via Helm values (`openclawConfig`)
- Preserve config changes made by the application at runtime
- Have both coexist without conflicts

The init container uses `jq` for deep merging with the formula: `.[0] * .[1]` (ConfigMap overlays PVC)

## Troubleshooting

### Pod stuck in Init:0/1
Check init container logs:
```bash
kubectl logs <pod-name> -c config-init
```

### Permission denied on config directory
Verify the security context user/group matches the image requirements. The image user must own `/home/node/.openclaw` and `/home/node/claw`.

### PVC not binding
Check available storage classes:
```bash
kubectl get storageclasses
```
The chart uses the default storage class if not specified.

## Building OpenClaw Image

OpenClaw doesn't have an official pre-built Docker image. Build using the Dockerfile:

```bash
git clone https://github.com/moltbot/openclaw.git
cd openclaw
docker build -t my-registry/openclaw:latest .
docker push my-registry/openclaw:latest
```

Then use in the chart:
```bash
helm install openclaw ./openclaw-chart \
  --set image.repository=my-registry/openclaw \
  --set image.tag=latest
```

## License

This Helm chart is provided as-is under the same license as OpenClaw.

## Contributing

Contributions are welcome! Please submit issues and pull requests to the [repository](https://github.com/HCLI-Hybrid-Cloud-Logistics-Initiatives/openclaw-helm-chart).
