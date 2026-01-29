# k8s-builder

## Purpose
Define reusable standards for Kubernetes manifests and platform configuration.

## Scope
Applies to Kubernetes YAML, Helm charts, and kustomize overlays in this repository.

## Standards

### Directory Design
- Keep Kubernetes assets under `kubernetes/` or within service-specific `deploy/` folders.
- Separate base manifests from environment overlays.
- Keep cluster-wide resources in a dedicated path.

### Manifests
- Use consistent labels and selectors.
- Define resource requests/limits for workloads.
- Configure health checks (readiness/liveness).
- Avoid default namespace; set explicit namespaces.

### Security
- Use least-privilege RBAC.
- Avoid embedding secrets in manifests; reference external secret stores when possible.
- Prefer pod security standards aligned with the cluster policy.

### Deployment Strategy
- Define rolling update parameters for workloads.
- Keep service discovery and ingress explicit.

### Compatibility
- Keep API versions current and avoid deprecated resources.
- Document breaking changes in the service README.
