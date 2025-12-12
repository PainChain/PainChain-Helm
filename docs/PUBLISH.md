# Publishing the PainChain Helm Chart

This document explains how the Helm chart is published and how to set up the CI/CD pipeline.

## Publishing Methods

The chart is published to **two locations** automatically when changes are merged to `main`:

### 1. GitHub Container Registry (GHCR) - OCI Format

**Registry**: `oci://ghcr.io/painchain/charts`

This is the **modern, recommended** approach for Helm charts using OCI (Open Container Initiative) format.

**Install from GHCR**:
```bash
helm install painchain oci://ghcr.io/painchain/charts/painchain --version 0.1.0
```

**Advantages**:
- Uses the same infrastructure as container images
- No need for separate index.yaml
- Better integration with existing registries
- Easier authentication

### 2. GitHub Pages - Traditional Helm Repository

**Repository URL**: `https://painchain.github.io/PainChain-Helm`

This is the traditional Helm chart repository format.

**Add the repository**:
```bash
helm repo add painchain https://painchain.github.io/PainChain-Helm
helm repo update
helm install painchain painchain/painchain
```

**Advantages**:
- Compatible with older Helm versions
- Familiar to users of traditional Helm repos
- Browsable via web interface

## CI/CD Workflows

### Testing Workflow (`.github/workflows/test.yaml`)

**Triggers**: Pull requests to `main`

**Steps**:
1. **Lint**: Uses `chart-testing` (ct) to lint the chart
2. **Template Validation**: Renders templates with `helm template`
3. **Manifest Validation**: Validates Kubernetes manifests with kubeval
4. **Install Test**: Creates a Kind cluster and tests chart installation
5. **Docs Check**: Verifies README is up to date

**Required for merge**: All checks must pass

### Release Workflow (`.github/workflows/release.yaml`)

**Triggers**: Push to `main` branch (after PR merge)

**Steps**:
1. **Package Chart**: Creates `.tgz` package
2. **Push to GHCR**: Pushes chart to OCI registry at `ghcr.io`
3. **Create GitHub Release**: Uses `chart-releaser` to create a GitHub release
4. **Update GitHub Pages**: Publishes to `gh-pages` branch for traditional Helm repo

## Setup Instructions

### Prerequisites

1. **GitHub Repository Settings**

   Enable GitHub Pages:
   - Go to Settings → Pages
   - Source: Deploy from a branch
   - Branch: `gh-pages` / `root`
   - Save

2. **GitHub Actions Permissions**

   Ensure Actions have necessary permissions:
   - Go to Settings → Actions → General
   - Workflow permissions: Read and write permissions
   - Allow GitHub Actions to create and approve pull requests: ✓

3. **Secrets** (automatically available)

   The workflows use `GITHUB_TOKEN` which is automatically provided by GitHub Actions.
   No manual secret configuration needed!

### First Release

1. **Update Chart Version**

   Edit `Chart.yaml`:
   ```yaml
   version: 0.1.0  # Increment this for each release
   appVersion: "0.1.0"  # Application version
   ```

2. **Create Initial gh-pages Branch**

   ```bash
   git checkout --orphan gh-pages
   git rm -rf .
   echo "# PainChain Helm Charts" > index.md
   git add index.md
   git commit -m "Initial gh-pages"
   git push origin gh-pages
   git checkout main
   ```

3. **Merge a PR**

   Once a PR is merged to main, the release workflow will:
   - Automatically create a GitHub release
   - Push the chart to GHCR
   - Update the GitHub Pages index

## Version Management

### Semantic Versioning

Follow [SemVer](https://semver.org/) for chart versions:

- **MAJOR** (e.g., 1.0.0 → 2.0.0): Breaking changes
- **MINOR** (e.g., 0.1.0 → 0.2.0): New features, backward compatible
- **PATCH** (e.g., 0.1.0 → 0.1.1): Bug fixes, backward compatible

### When to Bump Versions

**PATCH** (0.1.0 → 0.1.1):
- Bug fixes in templates
- Documentation updates
- Small configuration tweaks

**MINOR** (0.1.0 → 0.2.0):
- New optional features
- New configurable parameters
- Non-breaking changes to defaults

**MAJOR** (0.1.0 → 1.0.0):
- Breaking changes to values.yaml structure
- Removal of features
- Changes requiring manual migration

### Release Process

1. **Update version in `Chart.yaml`**
2. **Update `appVersion` if application changed**
3. **Update CHANGELOG.md** (if exists)
4. **Create PR with version bump**
5. **Merge PR** → Release workflow runs automatically
6. **Verify release**:
   - Check GitHub Releases page
   - Check GHCR packages
   - Test installation from both sources

## Testing Before Release

### Local Testing

```bash
# Lint the chart
helm lint .

# Render templates
helm template painchain . --debug

# Test installation in a test cluster
helm install painchain . --dry-run --debug

# Actual test install
helm install painchain . --namespace painchain-test --create-namespace

# Cleanup
helm uninstall painchain --namespace painchain-test
kubectl delete namespace painchain-test
```

### CI Testing

Push to a PR branch and the test workflow will automatically:
- Lint the chart
- Validate templates
- Test installation in a Kind cluster

## Publishing to Artifact Hub

Once your chart is published, you can list it on [Artifact Hub](https://artifacthub.io/):

1. Create an `artifacthub-repo.yml` file:
   ```yaml
   repositoryID: <your-repo-id>
   owners:
     - name: PainChain Team
       email: team@painchain.io
   ```

2. Add the repository on Artifact Hub dashboard
3. It will automatically index your GitHub Pages Helm repo

## Troubleshooting

### Release Workflow Fails

**Check permissions**:
```bash
# Ensure GITHUB_TOKEN has write access to packages
# Settings → Actions → General → Workflow permissions
```

**Check branch protection**:
```bash
# The workflow needs to push to gh-pages
# Settings → Branches → gh-pages should allow force pushes from Actions
```

### Chart Not Appearing in GHCR

1. Check the Actions tab for workflow failures
2. Verify package visibility: Settings → Packages → painchain → Change visibility to Public
3. Check the package exists: https://github.com/orgs/painchain/packages

### Chart Not in GitHub Pages

1. Check that `gh-pages` branch exists
2. Verify GitHub Pages is enabled in repository settings
3. Check the index.yaml file in gh-pages branch
4. Wait 1-2 minutes for Pages to rebuild

## Manual Publishing (Emergency)

If CI/CD fails, you can publish manually:

### To GHCR (OCI)
```bash
# Login
echo $GITHUB_TOKEN | helm registry login ghcr.io -u $GITHUB_USER --password-stdin

# Package and push
helm package .
helm push painchain-0.1.0.tgz oci://ghcr.io/painchain/charts
```

### To GitHub Pages
```bash
# Package chart
helm package .

# Checkout gh-pages
git checkout gh-pages

# Update index
helm repo index . --url https://painchain.github.io/PainChain-Helm

# Commit and push
git add .
git commit -m "Release painchain-0.1.0"
git push origin gh-pages

# Return to main
git checkout main
```

## Resources

- [Helm OCI Support](https://helm.sh/docs/topics/registries/)
- [Chart Releaser Action](https://github.com/helm/chart-releaser-action)
- [Artifact Hub](https://artifacthub.io/)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
