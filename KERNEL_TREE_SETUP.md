# Kdevops CI Setup for Linux Kernel Trees

This document explains how to add kdevops CI to your Linux kernel git tree with minimal configuration.

## Minimal Requirements

Your kernel tree needs only **one file** to enable kdevops CI:

### `.github/workflows/ci.yml`

```yaml
name: Kdevops CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: ['**']
  workflow_dispatch:  # Allow manual triggering

jobs:
  kdevops-workflow:
    uses: linux-kdevops/kdevops-ci/.github/workflows/kdevops.yml@unified-kpd-tests
    secrets: inherit
```

## Prerequisites

### 1. Repository Secrets

Configure these secrets in your repository settings:

- **`SSH_PRIVATE_KEY`**: Private SSH key for accessing kdevops infrastructure
  - Must have access to `kdevops-results-archive` repository
  - Should be in OpenSSH private key format

### 2. Kdevops Defconfig

Your repository name must have a corresponding defconfig in the kdevops repository:

- **File location**: `kdevops/defconfigs/YOUR_REPO_NAME`
- **Custom name**: Override with `kdevops_defconfig` input parameter

Example for custom defconfig:
```yaml
jobs:
  kdevops-workflow:
    uses: linux-kdevops/kdevops-ci/.github/workflows/kdevops.yml@unified-kpd-tests
    with:
      kdevops_defconfig: custom-config-name
    secrets: inherit
```

### 3. Self-Hosted Runners

The CI requires self-hosted runners with:
- Label: `[self-hosted, Linux, X64]`
- Access to kdevops infrastructure
- Docker support for containerized testing

## What the CI Does

1. **Setup Phase**:
   - Validates secrets and configuration
   - Clones your kernel tree with optimized mirroring
   - Runs quick defconfig build test
   - Sets up kdevops environment and DUT nodes
   - Builds kernel and installs on test nodes

2. **Test Phase**:
   - Runs `make ci-test` in kdevops environment
   - Executes tests defined by your defconfig

3. **Cleanup Phase** (always runs):
   - Collects systemd journal logs
   - Archives test results
   - Uploads artifacts to GitHub
   - Destroys test infrastructure

## Advanced Configuration

### Multiple Test Suites

```yaml
strategy:
  matrix:
    test_suite: [generic, fstests, blktests]
jobs:
  kdevops-workflow:
    uses: linux-kdevops/kdevops-ci/.github/workflows/kdevops.yml@unified-kpd-tests
    with:
      test_suite: ${{ matrix.test_suite }}
    secrets: inherit
```

### Conditional Triggers

```yaml
on:
  push:
    branches: ['master', 'testing/**']
    paths-ignore: ['Documentation/**', '*.md']
  pull_request:
    branches: ['master']
```

## Git History Handling

The CI intelligently handles two scenarios:

1. **Ephemeral .github commits** (e.g., kernel-patch-daemon):
   - If the last commit adds `.github/`, it tests the commit before it
   - Useful for testing mailing list patches

2. **Development trees**:
   - Tests the actual HEAD commit
   - Normal workflow for kernel development

## Troubleshooting

### Common Issues

- **Secret validation fails**: Ensure `SSH_PRIVATE_KEY` is configured
- **Defconfig not found**: Create `defconfigs/YOUR_REPO_NAME` in kdevops
- **Runner unavailable**: Check self-hosted runner status
- **Build failures**: Check kernel defconfig compatibility

### Debug Information

The CI provides detailed logs including:
- Git references and tags being tested
- Kdevops configuration parameters
- Host prefix for test isolation
- Full command invocations

## Production Deployment

For production use, consider:

1. **Use tagged versions instead of branch names**:
   ```yaml
   uses: linux-kdevops/kdevops-ci/.github/workflows/kdevops.yml@v1.0.0
   ```

2. **Pin runner requirements** in your defconfig
3. **Configure notification webhooks** for results
4. **Set up result archiving** to your preferred storage

## Examples

See these repositories for working examples:
- `linux-firmware-kpd`: Basic CI setup
