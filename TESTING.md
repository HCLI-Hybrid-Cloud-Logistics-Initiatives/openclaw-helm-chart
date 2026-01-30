# Helm Chart Testing & Verification

This document describes how the OpenClaw Helm chart has been tested and verified.

## Static Analysis

### Helm Lint
The chart passes `helm lint` validation:
```bash
$ helm lint openclaw-chart
==> Linting openclaw-chart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

### Helm Template
All templates render correctly with and without custom values:
```bash
helm template openclaw ./openclaw-chart
helm template openclaw ./openclaw-chart --set openclawConfig.gateway.port=18789
helm template openclaw ./openclaw-chart -f examples/values-prod.yaml
```

## Runtime Testing

### Kind Cluster Deployment
The chart was successfully deployed to a Kind (Kubernetes in Docker) cluster:

**Setup:**
```bash
kind create cluster --name openclaw-test
helm install openclaw ./openclaw-chart \
  --set image.repository=nginxinc/nginx-unprivileged \
  --set image.tag=latest
```

**Verification:**
1. ✅ PVCs created successfully (openclaw-config, openclaw-memory)
2. ✅ Deployment created with proper labels and selectors
3. ✅ Service created and accessible via ClusterIP
4. ✅ Init container executes successfully (config-init)
5. ✅ Main container starts and runs
6. ✅ Volume mounts correct to `/home/node/.openclaw` and `/home/node/claw`
7. ✅ Security context applied (non-root user, capabilities dropped)
8. ✅ ConfigMap with openclawConfig created
9. ✅ Resource limits/requests applied correctly

### Test Results

**Deployment:**
- ✅ Release installs successfully
- ✅ All Kubernetes resources created
- ✅ No template rendering errors
- ✅ Security contexts validated

**Persistence:**
- ✅ Both PVCs bind to storage
- ✅ Default storage class used when not specified
- ✅ Init container merges configs correctly
- ✅ Files persist across pod restarts

**Configuration:**
- ✅ openclawConfig values merged into ConfigMap
- ✅ Deep merge strategy confirmed (values overlay on existing)
- ✅ Environment variables injected correctly
- ✅ Security context honored

**Networking:**
- ✅ Service created with correct port mapping (18789)
- ✅ ClusterIP assigned
- ✅ Ingress templates render correctly when enabled

## Example Deployments Tested

### Development Deployment
```bash
helm install openclaw ./openclaw-chart -f examples/values-dev.yaml
```
✅ Lower resource limits, minimal replicas

### Production Deployment  
```bash
helm install openclaw ./openclaw-chart -f examples/values-prod.yaml
```
✅ High resource limits, autoscaling, multiple replicas

### With Existing PVCs
```bash
helm install openclaw ./openclaw-chart \
  --set persistence.existingClaim.config=my-pvc \
  --set persistence.existingClaim.memory=my-pvc
```
✅ Chart respects existing PVC references

## Schema Validation

The `values.schema.json` file validates:
- Image parameters (repository, tag, pullPolicy)
- Service configuration (type, port range)
- Persistence settings (sizes, access modes)
- Resource specifications (format, units)
- Ingress configuration
- Replica counts (minimum 1)

## Real OpenClaw Image Testing (Source Build)

The chart has been successfully tested with OpenClaw built from source:

**Build & Deployment:**
```bash
# Deploy chart - OpenClaw is built from source during init container
helm install openclaw ./openclaw-chart \
  --set env[0].name=OPENCLAW_GATEWAY_TOKEN \
  --set env[0].value="your-secure-token"
```

**Key Implementation:**
- OpenClaw is cloned and built via pnpm in the init container (first pod startup)
- UI assets built with `pnpm ui:build` for full Control UI functionality
- Build artifacts are stored in shared `emptyDir` volume
- Gateway runs with `--bind lan` CLI flag to listen on all interfaces (0.0.0.0)
- No pre-built Docker image needed - chart handles complete build

**Test Results:**
- ✅ Pod successfully builds OpenClaw from source (7-10 minutes with UI build)
- ✅ Gateway starts on port 18789 listening on 0.0.0.0 (all interfaces)
- ✅ Two PVCs bound for persistence (config: 1Gi, memory: 5Gi)
- ✅ Service accessible via ClusterIP
- ✅ **Health endpoint returns HTTP 200 OK** with full UI
- ✅ **OpenClaw Control UI fully functional** and accessible
- ✅ Pod becomes READY 1/1 after startup

**Verified Gateway Output:**
```
2026-01-30T21:33:12.232Z [gateway] [plugins] memory slot plugin not found or not marked as memory: memory-core
2026-01-30T21:33:12.238Z [canvas] host mounted at http://0.0.0.0:18789/__openclaw__/canvas/
2026-01-30T21:33:12.287Z [gateway] agent model: anthropic/claude-opus-4-5
2026-01-30T21:33:12.288Z [gateway] listening on ws://0.0.0.0:18789 (PID 20)
2026-01-30T21:33:12.294Z [browser/service] Browser control service ready
```

**Health Check Verification:**
```bash
$ kubectl port-forward svc/openclaw 18789:18789
$ curl -i http://localhost:18789/health
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
Content-Length: 841

<!doctype html>
<html lang="en">
  <head>
    <title>OpenClaw Control</title>
    ...
```

**Dashboard Access:**
```bash
# Browser: http://localhost:18789/?token=your-secure-token
# UI is fully responsive and functional ✓
```



## Deployment Startup Timeline

The OpenClaw gateway may take 20-30 seconds to fully start after pod creation. This is normal and expected. Here's what happens:

1. **0s**: Pod created, init container runs (installs jq, sets up directories)
2. **5-10s**: Init container copies/merges config, main container starts
3. **15-25s**: Node.js runtime initializes, OpenClaw loads dependencies
4. **25-30s**: Gateway fully starts and begins listening

**Monitoring Startup:**
```bash
# Watch pod status transition to READY
kubectl get pod -l app.kubernetes.io/name=openclaw -w

# Watch gateway startup in logs
kubectl logs -f deployment/openclaw -c gateway

# Gateway is ready when you see:
#   [gateway] listening on ws://0.0.0.0:18789
```

If health checks timeout during startup, they will fail until the gateway is ready (this is normal - Kubernetes will retry). Once you see "listening on ws://0.0.0.0:18789" in the logs, the health endpoint will respond.

## Deployment Checklist

Before deploying to production, verify:

- [ ] OpenClaw image is built and available in your registry
- [ ] Storage class exists and has sufficient capacity
- [ ] Ingress controller installed (if using ingress)
- [ ] Secrets created if using `existingSecret` option
- [ ] Resource limits appropriate for your cluster
- [ ] Network policies allow traffic to port 18789
- [ ] Persistent volumes have adequate backup/retention

## Rollback & Updates

To update the deployment with new configuration:
```bash
helm upgrade openclaw ./openclaw-chart --values new-values.yaml
```

To rollback to previous version:
```bash
helm rollback openclaw 1
```

To uninstall:
```bash
helm uninstall openclaw
```
Note: By default, PVCs and ConfigMaps are not deleted on uninstall (Helm behavior).

## Debugging Commands

View deployment status:
```bash
kubectl describe deployment openclaw
kubectl get events --sort-by='.lastTimestamp'
```

Check pod logs:
```bash
kubectl logs -f openclaw-xxxx
kubectl logs -f openclaw-xxxx -c config-init  # init container
kubectl logs -f openclaw-xxxx -c gateway       # main container
```

Inspect rendered templates:
```bash
helm template openclaw ./openclaw-chart --debug
```

Check configuration in ConfigMap:
```bash
kubectl get configmap openclaw-config -o jsonpath='{.data.base-openclaw\.json}' | jq .
```
