# docker-builder

## Purpose
Define reusable standards for Docker images and container build artifacts.

## Scope
Applies to Dockerfiles, container build scripts, and compose files in this repository.

## Standards

### Directory Design
- Keep Docker assets under a clear path such as `docker/` or alongside the service.
- Use one Dockerfile per image; name additional variants explicitly (e.g., `Dockerfile.dev`).
- Store build-time assets with the service; avoid shared global build context.

### Dockerfile
- Prefer multi-stage builds for slim runtime images.
- Pin base image tags to explicit versions.
- Use non-root users where possible.
- Keep layers minimal and cache-friendly.
- Avoid baking secrets into images.

### Local Container Networking
- Local containers must communicate only via Docker network DNS.
- Do not expose container ports externally.

### Image Metadata
- Use consistent labels (name, version, commit) when required.
- Document image purpose and expected runtime in the service README.

### Compose (if used)
- Keep environment overrides in separate files.
- Do not hardcode secrets in compose files.
- Avoid exposing container ports externally; use internal-only networking with `networks` and service-to-service DNS.

### Compatibility
- Keep image changes backward-compatible when possible.
- Document breaking runtime changes.
