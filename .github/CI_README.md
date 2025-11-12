# CI/CD Pipeline Documentation

This repository includes comprehensive GitHub Actions workflows to ensure compatibility with both Docker Compose and Podman Compose.

## Workflows Overview

### 1. Lint and Validate (`lint-and-validate.yml`)
**Trigger**: Push to main/develop/claude branches, PRs to main/develop

Runs multiple linting and validation checks:
- **YAML Lint**: Validates YAML syntax in compose files and workflows
- **Compose Validate**: Validates docker-compose.yml structure
- **Dockerfile Lint**: Uses Hadolint to check Dockerfile best practices
- **ShellCheck**: Validates shell scripts
- **Markdown Lint**: Checks markdown formatting

### 2. Docker Compose Tests (`docker-compose-test.yml`)
**Trigger**: Push to main/develop/claude branches, PRs to main/develop

Tests Docker Compose functionality:
- **Build Images**: Builds custom n8n image and validates configuration
- **Start Services**: Launches core services (Postgres, Qdrant) and tests connectivity
- **Security Scan**: Runs Trivy vulnerability scanner on configurations

**Matrix Testing**:
- CPU profile only (GPU requires runners with NVIDIA support)

### 3. Podman Compose Tests (`podman-compose-test.yml`)
**Trigger**: Push to main/develop/claude branches, PRs to main/develop

Tests Podman Compose compatibility:
- **Podman Build**: Builds images using Podman
- **Podman Services**: Tests service startup and health checks
- **Compatibility Check**: Validates compose file features work with Podman

**Key Differences from Docker**:
- Installs Podman and podman-compose from system packages
- Uses Podman-specific commands for health checks
- Tests rootless container execution

### 4. Full CI Pipeline (`ci-full.yml`)
**Trigger**: Push to main/develop branches, PRs to main/develop

Comprehensive end-to-end testing:
1. **Lint**: All validation checks
2. **Test Docker**: Full Docker Compose test suite
3. **Test Podman**: Full Podman Compose test suite
4. **Security**: Trivy scans on Dockerfile and configs
5. **Compatibility Matrix**: Generates compatibility report

**Artifacts**: Uploads compatibility report showing Docker/Podman test results

## Running Tests Locally

### Docker Tests
```bash
# Validate compose file
docker compose config --quiet

# Build images
docker compose build

# Start services
docker compose --profile cpu up -d

# Check health
docker compose ps
docker compose logs
```

### Podman Tests
```bash
# Install podman-compose
pip install podman-compose

# Validate compose file
podman-compose config --quiet

# Build images
podman-compose build

# Start services
podman-compose --profile cpu up -d

# Check health
podman ps -a
podman-compose logs
```

## Test Matrix

| Test | Docker | Podman | Status |
|------|--------|--------|--------|
| YAML Validation | ✅ | ✅ | Both |
| Image Build | ✅ | ✅ | Both |
| Postgres Startup | ✅ | ✅ | Both |
| Qdrant Startup | ✅ | ✅ | Both |
| Health Checks | ✅ | ✅ | Both |
| Security Scan | ✅ | ✅ | Both |
| GPU Profile | ⚠️ | ⚠️ | Requires GPU runner |

## Profiles Tested

- **cpu**: CPU-only vLLM service (default for CI)
- **gpu**: GPU-accelerated vLLM (requires NVIDIA GPU runner)

*Note: GPU tests are not run in CI by default due to runner requirements*

## What Gets Tested

### Build Phase
- ✅ n8n custom Dockerfile builds successfully
- ✅ All service images can be pulled
- ✅ Compose file syntax is valid
- ✅ No conflicting port definitions

### Runtime Phase
- ✅ PostgreSQL starts and becomes healthy
- ✅ Qdrant starts successfully
- ✅ Database connections work
- ✅ Data directories are created correctly
- ✅ Network connectivity between services

### Security Phase
- ✅ No critical vulnerabilities in Dockerfile
- ✅ No misconfigurations in compose file
- ✅ Proper permission handling

## CI/CD Best Practices

### What's Included
1. **Multi-tool testing**: Both Docker and Podman validated
2. **Security scanning**: Trivy checks for vulnerabilities
3. **Linting**: YAML, Dockerfile, Shell, and Markdown validation
4. **Health checks**: Services verified to be running correctly
5. **Artifact generation**: Compatibility reports uploaded

### What's Not Included (Yet)
- Full integration tests with all services
- GPU runner tests
- Performance benchmarking
- End-to-end workflow testing
- Load testing

## Troubleshooting CI Failures

### Lint Failures
- Check YAML indentation (use 2 spaces)
- Ensure no trailing whitespace
- Validate Dockerfile instructions

### Docker Test Failures
- Check for port conflicts
- Verify image tags are available
- Review service dependencies

### Podman Test Failures
- Podman may have stricter security defaults
- Check for Docker-specific features
- Verify rootless compatibility

### Security Scan Failures
- Update base images to latest versions
- Review Dockerfile for security best practices
- Check for exposed secrets or credentials

## Adding New Services

When adding new services to docker-compose.yml:

1. Add to appropriate workflow if it needs testing
2. Update health checks in CI workflows
3. Add to compatibility matrix documentation
4. Test with both Docker and Podman locally first

Example:
```yaml
# In ci-full.yml
- name: Start new service
  run: docker compose up -d new-service

- name: Test new service
  run: curl -f http://localhost:PORT/health
```

## Status Badges

Add these to your README.md:

```markdown
![Lint](https://github.com/USERNAME/REPO/workflows/Lint%20and%20Validate/badge.svg)
![Docker Tests](https://github.com/USERNAME/REPO/workflows/Docker%20Compose%20Tests/badge.svg)
![Podman Tests](https://github.com/USERNAME/REPO/workflows/Podman%20Compose%20Tests/badge.svg)
![Full CI](https://github.com/USERNAME/REPO/workflows/Full%20CI%20Pipeline/badge.svg)
```

## Contributing

When contributing, ensure:
- All CI checks pass
- Both Docker and Podman tests succeed
- No new security vulnerabilities introduced
- Documentation is updated

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Podman Compose Documentation](https://github.com/containers/podman-compose)
- [Hadolint Rules](https://github.com/hadolint/hadolint)
- [Trivy Scanner](https://github.com/aquasecurity/trivy)
