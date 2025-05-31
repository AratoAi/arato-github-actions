# Arato GitHub Action

A comprehensive GitHub Action for building and monitoring Arato AI experiments with automatic progress tracking.

## 🚀 Quick Start for Users

### 1. Use in Your Workflow

Add this to your `.github/workflows/build.yml`:

```yaml
name: Build AI Models
on: [workflow_dispatch]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build with Arato
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: 'my-flow/my-experiment'
          api_keys: '{"openai_api_key": "${{ secrets.OPENAI_API_KEY }}"}'
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
```

### 2. Add Required Secrets

Go to your repository **Settings → Secrets → Actions** and add:

- `ARATO_API_KEY` - Your Arato API key (starts with `ar-`)
- `OPENAI_API_KEY` - Your OpenAI API key (if using OpenAI models)
- `ANTHROPIC_API_KEY` - Your Anthropic API key (if using Claude models)

### 3. Run Your Build

1. Go to **Actions** tab
2. Select your workflow
3. Click **Run workflow**
4. Your build will start automatically with monitoring!

## 📋 Configuration Options

### Input Parameters

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| `experiments` | ✅ Yes | Experiment(s) to build | `"flow/exp"` or `["flow/exp1", "flow/exp2"]` |
| `api_keys` | ✅ Yes | AI model API keys as JSON | `{"openai_api_key": "sk-..."}` |
| `api_base_url` | ❌ No | Custom Arato API endpoint | `https://staging.arato.ai` |

### Output Values

| Output | Type | Description |
|--------|------|-------------|
| `success` | boolean | Whether all builds completed successfully |
| `completed_count` | number | Number of successful experiments |
| `failed_count` | number | Number of failed experiments |
| `results_summary` | JSON | Detailed build results |

## 💡 Usage Examples

### Single Experiment
```yaml
with:
  experiments: 'customer-support/sentiment-analysis'
  api_keys: '{"openai_api_key": "${{ secrets.OPENAI_API_KEY }}"}'
```

### Multiple Experiments
```yaml
with:
  experiments: '["flow1/exp1", "flow1/exp2", "flow2/exp1"]'
  api_keys: |
    {
      "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
      "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"
    }
```

### With Result Processing
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build with Arato
        id: arato-build
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: '${{ inputs.experiments }}'
          api_keys: '${{ inputs.api_keys }}'
          arato_api_key: ${{ secrets.ARATO_API_KEY }}

      - name: Send notification
        run: |
          if [[ "${{ steps.arato-build.outputs.success }}" == "true" ]]; then
            echo "✅ Build completed successfully!"
          else
            echo "❌ Build failed. Check logs for details."
          fi
          fi
```

## 🔧 Advanced Features

### Automatic Monitoring
- ✅ Polls build status every 30 seconds
- ✅ Automatically detects completion
- ✅ 60-minute timeout protection
- ✅ Real-time progress updates

### Rich Reporting
- 📊 GitHub Actions summary with results
- 📄 Downloadable build reports
- 🔍 Detailed logs for debugging
- 📈 Success/failure metrics

### Enterprise Ready
- 🔒 Secure API key handling
- 🌐 Custom API endpoints
- 🔄 Version pinning support
- ⚡ Optimized for CI/CD pipelines

## 🚨 Troubleshooting

### Common Issues

**❌ "Invalid JSON format for API keys"**
- Ensure your `api_keys` parameter is valid JSON
- Use proper quotes: `{"key": "value"}`

**❌ "Build failed with HTTP 401"**
- Check your `ARATO_API_KEY` secret is correct
- Ensure the key starts with `ar-`

**⏰ "Build timeout"**
- This is normal for complex experiments
- Results will show current status
- Check Arato dashboard for final results

## 📚 Complete Examples

Check our [examples directory](examples/) for full workflow templates:

- **[Basic Usage](examples/basic-usage.md)** - Simple single experiment
- **[Multiple Experiments](examples/multiple-experiments.md)** - Batch processing
- **[CI/CD Pipeline](examples/cicd-pipeline.md)** - Automated deployments

## 🔄 Version Management

### Recommended Usage
```yaml
# Production (recommended)
uses: AratoAi/arato@v1.0.0

# Latest features
uses: AratoAi/arato@main
```

## 🤝 Support

- 📖 **Documentation** - Check the [README](README-GITHUB-ACTION.md) and examples
- 🐛 **Issues** - Report problems in our Issues tab
- 💬 **Discussions** - Ask questions in GitHub Discussions
- 📧 **Contact** - Reach out to the Arato team

---

**Start building smarter AI workflows today!** 🚀

### Basic Usage

Other repositories can use your workflow by creating a workflow file like this:

```yaml
# .github/workflows/use-arato-build.yml
name: Build with Arato

on:
  workflow_dispatch:
    inputs:
      experiments:
        description: 'Experiments to build'
        required: true
        type: string
        default: '["flow_id/experiment_id"]'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build experiments
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ inputs.experiments }}
          api_keys: ${{ secrets.API_KEYS }}
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
```

### Advanced Usage

```yaml
# .github/workflows/advanced-arato-build.yml
name: Advanced Arato Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Build and test experiments
        uses: AratoAi/arato@v1.0.0
        with:
          api_base_url: 'https://staging.arato.ai'  # Custom API endpoint
          experiments: '["my-flow/experiment-1", "my-flow/experiment-2"]'
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
    secrets:
      ARATO_API_KEY: ${{ secrets.ARATO_API_KEY }}

  post-build:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Report Results
        run: |
          echo "Build Status: ${{ needs.build-and-test.outputs.build_status }}"
          echo "Completed: ${{ needs.build-and-test.outputs.completed_count }}"
          echo "Failed: ${{ needs.build-and-test.outputs.failed_count }}"
```

## 📋 Required Setup for Users

### 1. Repository Secrets

Users need to configure these secrets in their repository:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `ARATO_API_KEY` | Arato API authentication key | `ar-xxxxxxxxxxxxxxxx` |
| `OPENAI_API_KEY` | OpenAI API key (if used) | `sk-xxxxxxxxxxxxxxxx` |
| `ANTHROPIC_API_KEY` | Anthropic API key (if used) | `ant-xxxxxxxxxxxxxxxx` |

### 2. Input Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `api_base_url` | string | No | `https://api.arato.ai` | Base URL of Arato API |
| `experiments` | string | Yes | - | JSON array or single experiment ID |
| `api_keys` | string | Yes | - | JSON object with API keys |

### 3. Output Parameters

| Output | Type | Description |
|--------|------|-------------|
| `build_status` | string | HTTP status code from build API |
| `experiment_ids` | string | JSON array of created experiment IDs |
| `completed_count` | string | Number of successful experiments |
| `failed_count` | string | Number of failed experiments |

## 🎯 Example Repository Setup

Create this template for users:

```yaml
# .github/workflows/arato-build.yml
name: Arato Build Pipeline

on:
  workflow_dispatch:
    inputs:
      experiments:
        description: 'Experiments to build (JSON array or single ID)'
        required: true
        type: string
        default: '["my-flow/my-experiment"]'

jobs:
  arato-build:
    name: Build and Monitor
    runs-on: ubuntu-latest
    steps:
      - name: Build experiments
        id: build
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ inputs.experiments }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}

      - name: Send notification
        if: always()
        run: |
          if [[ "${{ steps.build.outputs.success }}" == "true" ]]; then
            echo "✅ Build successful! Completed: ${{ needs.arato-build.outputs.completed_count }}"
          else
            echo "❌ Build failed with status: ${{ needs.arato-build.outputs.build_status }}"
          fi
```

## 📖 Documentation Template

Create a README.md for users:

```markdown
# Arato Build Workflow

A reusable GitHub Actions workflow for triggering and monitoring Arato builds.

## Features

- ✅ Trigger builds via Arato API
- 🔍 Monitor build progress (30s polling, 60m timeout)
- 📊 Generate comprehensive reports
- 🔒 Secure API key handling
- 📈 Track success/failure metrics

## Quick Start

1. Add required secrets to your repository
2. Create a workflow file using our template
3. Run the workflow with your experiments

## Usage

See [examples/](examples/) for complete usage examples.
```

## 🔄 Version Management

### Semantic Versioning

Use semantic versioning for releases:
- `v1.0.0` - Major release
- `v1.1.0` - Minor features
- `v1.0.1` - Bug fixes

### Branching Strategy

- `main` - Stable, production-ready
- `develop` - Latest features
- `feature/*` - Feature branches

Users can reference specific versions:
```yaml
uses: AratoAi/arato@v1.0.0  # Specific version
uses: AratoAi/arato@main    # Latest stable
```

## 🛠️ Maintenance

### Breaking Changes

When making breaking changes:
1. Create a new major version (e.g., `v2.0.0`)
2. Update documentation
3. Provide migration guide
4. Maintain backward compatibility when possible

### Support

- Document known issues
- Provide troubleshooting guide
- Set up issue templates
- Consider creating discussions for Q&A

## 🚀 Next Steps

1. **Test the action** in a separate repository
2. **Gather user feedback** and iterate on documentation  
3. **Set up automated testing** for the action
4. **Create release versions** with semantic versioning (v1.0.0, v1.1.0, etc.)
5. **Publish to GitHub Marketplace** for wider discoverability

This approach makes your Arato build action easily consumable by other teams and organizations while maintaining centralized control and updates.

## 🚀 Publishing to GitHub Marketplace

### Prerequisites for Publishing

1. **Repository Setup**
   - Repository must be public
   - Must have a valid `action.yml` file in the root
   - Repository name should be descriptive (e.g., `arato-build-action`)

2. **Action Metadata**
   - Clear `name` and `description` in `action.yml`
   - Appropriate `branding` with icon and color
   - Comprehensive documentation in README

3. **Version Management**
   - Use semantic versioning (e.g., `v1.0.0`)
   - Tag releases properly
   - Maintain backward compatibility

### Publishing Steps

1. **Prepare the Repository**
   ```bash
   # Ensure action.yml is in the root
   # Ensure README-GITHUB-ACTION.md is comprehensive
   # Add examples/ directory with usage templates
   ```

2. **Create a Release**
   - Go to GitHub repository → Releases → Create new release
   - Use semantic version tag (e.g., `v1.0.0`)
   - Write clear release notes describing features
   - Check "Publish this action to the GitHub Marketplace"

3. **Marketplace Listing**
   - Choose appropriate categories (e.g., "Continuous Integration", "Machine Learning")
   - Provide clear description and keywords
   - Upload a logo/icon if available

### Post-Publishing Maintenance

- Monitor issues and user feedback
- Keep documentation updated
- Respond to community questions
- Maintain version compatibility
