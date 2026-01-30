# Contributing to OpenClaw Helm Chart

Thank you for your interest in contributing to the OpenClaw Helm Chart! This is a community-driven project to make deploying OpenClaw on Kubernetes easier and more accessible.

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Create a new branch for your changes: `git checkout -b feature/your-feature-name`

## Making Changes

### Chart Structure
- Keep templates in `openclaw-chart/templates/`
- Update `openclaw-chart/values.yaml` with new defaults
- Update `openclaw-chart/values.schema.json` for validation
- Add helper templates to `openclaw-chart/templates/_helpers.tpl`
- Update the `Chart.yaml` version if making changes

### Testing Your Changes

1. Lint the chart:
```bash
helm lint ./openclaw-chart
```

2. Render templates to verify syntax:
```bash
helm template openclaw ./openclaw-chart
```

3. Test with different values files:
```bash
helm template openclaw ./openclaw-chart -f examples/values-dev.yaml
helm template openclaw ./openclaw-chart -f examples/values-prod.yaml
```

4. Deploy to a test cluster (Kind/Minikube):
```bash
helm install openclaw ./openclaw-chart
kubectl get pods -w
kubectl logs -f <pod-name>
```

5. Test hot-reload functionality:
```bash
kubectl exec -it <pod-name> -- openclaw gateway restart
```

### Documentation

- Update `README.md` when adding new features or configuration options
- Add configuration examples to `examples/` directory
- Update `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/) format
- Document any new init container or deployment behavior

## Pull Request Process

1. **Test thoroughly** - Ensure `helm lint` passes with zero errors
2. **Update docs** - Update README.md, CHANGELOG.md, and examples as needed
3. **Test edge cases** - Test with disabled features, existing PVCs, different persistence configs
4. **Create PR** with clear description of:
   - What problem does this solve?
   - What changed and why?
   - How can reviewers test this?
5. **Link issues** - Reference any related GitHub issues

## Reporting Issues

When reporting issues, please include:
- Helm version: `helm version`
- Kubernetes version: `kubectl version`
- Chart version: `helm show chart ./openclaw-chart | grep version`
- Steps to reproduce
- Expected vs actual behavior
- Pod logs: `kubectl logs <pod-name>`
- Chart rendering output: `helm template openclaw ./openclaw-chart`

## Code Style

- Use 2-space indentation in YAML files
- Follow [Helm best practices](https://helm.sh/docs/chart_best_practices/)
- Keep templates readable and well-commented where logic isn't obvious
- Test both enabled and disabled features
- Ensure backward compatibility with previous versions

## Areas for Contribution

We welcome contributions in these areas:
- Bug fixes and improvements
- Documentation improvements
- New Helm chart examples
- Security hardening
- Performance optimizations
- Additional init container features
- Better error messages and validation

## Release Process

Releases are automated via GitHub Actions:
1. Update `Chart.yaml` version
2. Update `CHANGELOG.md` with changes
3. Tag the release: `git tag v0.x.x`
4. Push tag: `git push origin v0.x.x`
5. GitHub Actions automatically:
   - Lints the chart
   - Packages it
   - Pushes to GHCR OCI registry
   - Creates a GitHub Release

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

By contributing, you agree that your contributions will be licensed under the MIT License.

## Questions or Ideas?

- **Questions**: Create a [Discussion](https://github.com/HCLI-Hybrid-Cloud-Logistics-Initiatives/openclaw-helm-chart/discussions)
- **Bug reports**: Create an [Issue](https://github.com/HCLI-Hybrid-Cloud-Logistics-Initiatives/openclaw-helm-chart/issues)
- **Features**: Discuss in Issues before starting work

Thank you for contributing! 🎉
