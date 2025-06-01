# GitHub Actions Workflows

This directory contains a simplified GitHub Actions workflow for managing the Arato GitHub Action releases.

## Release Workflow

### 🚀 [`release.yml`](release.yml) - Automated Release

**Triggers:**
- Push of version tags (e.g., `v1.0.0`, `v2.1.3`)

**What it does:**
1. **Validates** the action.yml syntax and required fields
2. **Tests** the action with mock endpoints to ensure structure works
3. **Creates** a GitHub release with auto-generated changelog
4. **Updates** major version tags (e.g., `v1`) and minor version tags (e.g., `v1.0`) to point to latest release

**Usage:**
```bash
# Create and push a version tag to trigger a release
git tag v1.0.0
git push origin v1.0.0
```

**What gets tested:**
- ✅ YAML syntax validation
- ✅ Required fields presence (name, description, inputs, outputs, runs)
- ✅ Required inputs exist (experiments, api_keys, arato_api_key)
- ✅ Action execution (with mock endpoints)
- ✅ Version tag management

**Automatic tag updates:**
- `v1.0.0` → creates/updates `v1` and `v1.0`
- `v2.1.5` → creates/updates `v2` and `v2.1`

This allows users to reference the action with:
- `uses: username/arato-github-actions@v1.0.0` (specific version)
- `uses: username/arato-github-actions@v1` (latest v1.x.x)
- `uses: username/arato-github-actions@v1.0` (latest v1.0.x)

## Simplified Architecture

This setup eliminates the need for:
- ❌ Separate CI workflows
- ❌ Pull request validation workflows  
- ❌ Local scripts directory
- ❌ Complex workflow interdependencies

Everything needed for releases is contained in a single workflow that only runs when you're ready to publish a new version.

## 📋 Release Process

1. **Prepare the release:**
   ```bash
   # Ensure your changes are committed and pushed
   git checkout main
   git pull origin main
   ```

2. **Create and push version tag:**
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

3. **Monitor the release workflow:**
   - Go to Actions tab in GitHub
   - Watch the "Release" workflow
   - Verify release is created successfully with all tests passing

The workflow automatically handles version tag management and creates a comprehensive release with changelog.

## 🆘 Troubleshooting

**Release workflow fails:**
- Check tag format (must be `vX.Y.Z`)
- Ensure action.yml is valid
- Verify all required inputs are present in action.yml

**Version tags not updating:**
- Check workflow logs for git permission issues
- Manual tag update: `git tag -f v1 v1.0.0 && git push -f origin v1`
