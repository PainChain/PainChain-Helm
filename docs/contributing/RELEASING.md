# Release Guide

This document explains how to release the PainChain Helm chart.

## Two Release Tracks

### 1. Bleeding Edge (main)

**Automatic on merge to main**

Every merge to `main` automatically publishes a bleeding-edge chart:
- **Version**: `0.0.0-main`
- **Tag**: `main` (like Docker images)
- **Use case**: Testing latest changes before official release

**Install bleeding edge**:
```bash
helm install painchain oci://ghcr.io/painchain/charts/painchain --version 0.0.0-main
```

### 2. Stable Release (tagged)

**Manual tag push for stable releases**

When ready to cut an official release:

1. **Update Chart.yaml version**:
   ```bash
   # Edit Chart.yaml
   version: 0.2.0  # Bump version
   appVersion: "0.2.0"  # Match application version
   ```

2. **Commit the version bump**:
   ```bash
   git add Chart.yaml
   git commit -m "Bump version to 0.2.0"
   git push origin main
   ```

3. **Create and push a tag**:
   ```bash
   git tag v0.2.0
   git push origin v0.2.0
   ```

4. **Automatic release happens**:
   - GitHub Actions workflow triggers
   - Verifies Chart.yaml version matches tag
   - Publishes to GHCR as `painchain:0.2.0`
   - Creates GitHub Release with notes
   - Attaches packaged chart (.tgz)

**Install stable release**:
```bash
helm install painchain oci://ghcr.io/painchain/charts/painchain --version 0.2.0
```

## Workflow Summary

```
┌─────────────────┐
│  Merge to main  │
└────────┬────────┘
         │
         ▼
   Publish 0.0.0-main
   (bleeding edge)


┌─────────────────┐
│  Push tag v0.2.0│
└────────┬────────┘
         │
         ▼
   ┌──────────────────┐
   │ Verify version   │
   │ matches tag      │
   └────────┬─────────┘
            │
            ▼
   Publish 0.2.0
   (stable release)
   Create GitHub Release
```

## Version Guidelines

Follow [Semantic Versioning](https://semver.org/):

- **0.1.0 → 0.1.1** (PATCH): Bug fixes, docs
- **0.1.0 → 0.2.0** (MINOR): New features, backward compatible
- **0.9.0 → 1.0.0** (MAJOR): Breaking changes

## Pre-Release Checklist

Before pushing a version tag:

- [ ] All tests pass in CI
- [ ] Chart.yaml version updated
- [ ] Chart.yaml appVersion updated (if app changed)
- [ ] README.md is current
- [ ] Test installation locally:
  ```bash
  helm install painchain . --dry-run --debug
  ```

## Examples

### Patch Release (Bug Fix)

```bash
# Update Chart.yaml version: 0.1.0 → 0.1.1
git add Chart.yaml
git commit -m "Fix pod selector label in deployment"
git push origin main

# Create release
git tag v0.1.1
git push origin v0.1.1
```

### Minor Release (New Feature)

```bash
# Update Chart.yaml version: 0.1.0 → 0.2.0
git add Chart.yaml
git commit -m "Add support for external Redis"
git push origin main

# Create release
git tag v0.2.0
git push origin v0.2.0
```

### Major Release (Breaking Change)

```bash
# Update Chart.yaml version: 0.9.0 → 1.0.0
git add Chart.yaml
git commit -m "BREAKING: Restructure values.yaml for consistency"
git push origin main

# Create release with breaking change notes
git tag v1.0.0
git push origin v1.0.0
```

## Troubleshooting

### Tag workflow fails with version mismatch

**Error**: "Chart.yaml version does not match tag version"

**Fix**: Ensure Chart.yaml version matches the tag (without the 'v' prefix)
```bash
# If tag is v0.2.0
# Chart.yaml should have: version: 0.2.0
```

### Chart not appearing in GHCR

1. Check GitHub Actions workflow run
2. Verify package visibility is "Public" in GitHub settings
3. Check: https://github.com/orgs/painchain/packages

### Need to delete a bad release

```bash
# Delete tag locally and remotely
git tag -d v0.2.0
git push origin :refs/tags/v0.2.0

# Delete GitHub Release (via web UI or gh CLI)
gh release delete v0.2.0
```

Then fix the issue and re-release with the same version.

## Testing Releases

### Test bleeding edge locally

```bash
# Pull and install
helm install painchain oci://ghcr.io/painchain/charts/painchain --version 0.0.0-main -n test --create-namespace

# Upgrade to test main changes
helm upgrade painchain oci://ghcr.io/painchain/charts/painchain --version 0.0.0-main -n test
```

### Test stable release locally

```bash
# Pull and install specific version
helm install painchain oci://ghcr.io/painchain/charts/painchain --version 0.2.0 -n test --create-namespace

# List available versions
helm search repo painchain --versions
```

## FAQ

**Q: Can I re-release the same version?**

A: No, GHCR doesn't allow overwriting. Delete the tag, fix the issue, and release a new patch version.

**Q: Should I delete main tag after release?**

A: No, main tag is continuously updated and represents the latest development version.

**Q: What if I forget to update Chart.yaml before tagging?**

A: The workflow will fail with a clear error. Delete the tag, update Chart.yaml, and re-push the tag.

**Q: Can users install "latest"?**

A: No, Helm requires explicit versions. Users can use `0.0.0-main` for latest development or specific versions for stable releases.
