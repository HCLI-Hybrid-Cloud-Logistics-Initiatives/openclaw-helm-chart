# Contributing to OpenClaw Helm Chart

Thank you for your interest in contributing to the OpenClaw Helm Chart!

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Create a new branch for your changes

## Making Changes

### Chart Structure
- Keep templates in `openclaw-chart/templates/`
- Update `openclaw-chart/values.yaml` with new defaults
- Update `openclaw-chart/values.schema.json` for validation
- Add helper templates to `openclaw-chart/templates/_helpers.tpl`

### Testing Your Changes

1. Lint the chart:
```bash
helm lint openclaw-chart
```

2. Render templates to verify syntax:
```bash
helm template openclaw ./openclaw-chart
```

3. Test with custom values:
```bash
helm template openclaw ./openclaw-chart -f examples/values-dev.yaml
```

4. Deploy to a test cluster (Kind/Minikube):
```bash
helm install openclaw ./openclaw-chart
kubectl get pods
```

### Documentation

- Update `README.md` when adding new features or parameters
- Add configuration examples to `examples/`
- Update `TESTING.md` if modifying test procedures
- Document any new init container behavior

## Pull Request Process

1. Test your changes thoroughly
2. Update documentation as needed
3. Run `helm lint` and verify no errors
4. Submit PR with clear description of changes
5. Link any related issues

## Reporting Issues

When reporting issues, please include:
- Helm version: `helm version`
- Kubernetes version: `kubectl version`
- Chart version being used
- Steps to reproduce
- Expected vs actual behavior

## Code Style

- Use 2-space indentation in YAML files
- Follow Helm best practices and conventions
- Keep templates readable and well-commented
- Test edge cases (e.g., disabled features, existing PVCs)

## Feature Ideas

Areas for future enhancement:
- Chromium sidecar container
- StatefulSet variant for more complex deployments
- Horizontal Pod Autoscaler templates
- NetworkPolicy templates
- PodDisruptionBudget
- ServiceMonitor for Prometheus integration
- ConfigMap reload mechanisms (e.g., using Reloader)

## License

By contributing, you agree that your contributions will be licensed under the same license as the OpenClaw project.

## Questions?

Create an issue or discussion in the repository with your questions.

Thank you for contributing! 🎉
