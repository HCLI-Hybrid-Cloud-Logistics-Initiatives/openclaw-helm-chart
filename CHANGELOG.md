# Changelog

All notable changes to the OpenClaw Helm Chart will be documented in this file.

## [0.0.1] - 2026-01-31

### Added
- Initial release of OpenClaw Helm chart
- StatefulSet deployment with persistent storage for config (`/home/node/.openclaw`) and extraction memory (`/home/node/claw`)
- Automatic OpenClaw build from source at pod startup (via init container)
- Service and optional Ingress configuration with WebSocket support
- Hot-reload support: modify config and restart gateway without pod restart using `openclaw gateway restart`
- Security hardened: dropped capabilities, security policies
- Comprehensive configuration via `values.yaml` with sensible defaults
- JSON schema validation for chart configuration
- Example values files for dev, production, and secrets-based deployments
- Detailed documentation and troubleshooting guide
- GitHub Actions workflow for automated OCI registry releases

### Known Limitations
- Single instance only (StatefulSet with replicas=1)
- OpenClaw is stateful and not designed for horizontal scaling
- Requires 5-7 minutes for initial pod startup (builds from source)
- Configuration primarily via `openclaw onboard` wizard after first deployment

### Installation
```bash
helm install openclaw oci://ghcr.io/hcli-hybrid-cloud-logistics-initiatives/openclaw-helm-chart --version 0.0.1
```

Or from source:
```bash
helm install openclaw ./openclaw-chart
```
