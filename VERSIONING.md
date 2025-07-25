# Kdevops CI Versioning Strategy

## Current State

The `unified-kpd-tests` branch serves as the development/testing branch for the unified CI approach.

## Recommended Production Strategy

### 1. Semantic Versioning Tags

Once stable, create semantic version tags:

```bash
git tag -a v1.0.0 -m "Initial unified kdevops CI release"
git push origin v1.0.0
```

Kernel trees should then reference tags instead of branches:

```yaml
uses: linux-kdevops/kdevops-ci/.github/workflows/kdevops.yml@v1.0.0
```

### 2. Branch Lifecycle

- **`unified-kpd-tests`**: Development and testing
- **`main`**: Stable releases only  
- **`v1.x`**: Maintenance branches for major versions

### 3. Breaking Changes

Major version bumps (v1.x → v2.x) for:
- Workflow input/output changes
- Required secret modifications  
- Infrastructure requirement changes

Minor version bumps (v1.0 → v1.1) for:
- New optional features
- Bug fixes
- Performance improvements

### 4. Migration Strategy

1. **Phase 1**: Continue using `@unified-kpd-tests` for testing
2. **Phase 2**: Tag v1.0.0 when stable, encourage tag usage
3. **Phase 3**: Deprecate branch references in favor of tags

### 5. Documentation Updates

Update `KERNEL_TREE_SETUP.md` examples to use tags once v1.0.0 is released.