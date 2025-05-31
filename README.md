# Arato Build GitHub Action

A GitHub Action for triggering and monitoring Arato AI builds with automatic progress tracking and comprehensive reporting.

## ✨ Features

- 🚀 **Easy Setup** - Just provide your experiments and API keys
- 🔍 **Auto-Monitoring** - Tracks build progress every 30 seconds
- 📊 **Rich Reports** - Detailed success/failure reporting
- 🔒 **Secure** - API keys handled through GitHub Secrets
- ⚡ **Fast** - Optimized for quick feedback

## 🚀 Quick Start

### 1. Add to Your Workflow

Create `.github/workflows/arato-build.yml` in your repository:

```yaml
name: Build AI Models

on:
  workflow_dispatch:
    inputs:
      experiments:
        description: 'Experiments to build'
        required: true
        default: 'my-flow/my-experiment'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build with Arato
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ inputs.experiments }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
```

### 2. Configure Secrets

Add these secrets to your repository (Settings → Secrets → Actions):

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `ARATO_API_KEY` | Your Arato API key | `ar-1234567890abcdef...` |
| `OPENAI_API_KEY` | OpenAI API key (if used) | `sk-1234567890abcdef...` |
| `ANTHROPIC_API_KEY` | Anthropic API key (if used) | `ant-1234567890abcdef...` |

### 3. Run Your Build

1. Go to **Actions** tab in your repository
2. Select "Build AI Models" workflow
3. Click "Run workflow"
4. Enter your experiment ID (e.g., `my-flow/my-experiment`)
5. Click "Run workflow"

## 📋 Configuration

### Inputs

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `experiments` | ✅ Yes | - | Single experiment or JSON array of experiments |
| `api_keys` | ✅ Yes | - | JSON object with your AI model API keys |
| `api_base_url` | ❌ No | `https://api.arato.ai` | Arato API endpoint |

### Outputs

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
      "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}",
      "together_api_key": "${{ secrets.TOGETHER_API_KEY }}"
    }
```

### Custom API Endpoint

```yaml
with:
  experiments: 'my-flow/my-experiment'
  api_base_url: 'https://staging.arato.ai'
  api_keys: '{"openai_api_key": "${{ secrets.OPENAI_API_KEY }}"}'
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
        if: always()
        run: |
          if [[ "${{ steps.arato-build.outputs.success }}" == "true" ]]; then
            echo "✅ All ${{ steps.arato-build.outputs.completed_count }} experiments completed!"
          else
            echo "❌ Build failed. ${{ steps.arato-build.outputs.failed_count }} experiments failed."
          fi
```

## 🔧 Advanced Configuration

### Conditional Builds

```yaml
name: Smart Build Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      experiments: ${{ steps.detect.outputs.experiments }}
    steps:
      - uses: actions/checkout@v4
      - name: Detect changed experiments
        id: detect
        run: |
          # Your logic to detect which experiments changed
          echo 'experiments=["flow1/exp1", "flow2/exp2"]' >> $GITHUB_OUTPUT

  build:
    needs: changes
    if: needs.changes.outputs.experiments != '[]'
    runs-on: ubuntu-latest
    steps:
      - name: Build changed experiments
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ needs.changes.outputs.experiments }}
          api_keys: '${{ vars.DEFAULT_API_KEYS }}'
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
```

### Matrix Builds

```yaml
jobs:
  build:
    strategy:
      matrix:
        environment: [staging, production]
        experiment: [sentiment-analysis, text-classification]
    runs-on: ubuntu-latest
    steps:
      - name: Build experiment
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: 'flows/${{ matrix.experiment }}'
          api_base_url: 'https://${{ matrix.environment }}.arato.ai'
          api_keys: '${{ secrets.API_KEYS }}'
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
```

## 📊 Monitoring & Reports

### What You Get

- **Real-time Progress** - See build status updates every 30 seconds
- **Visual Summary** - Clear success/failure indicators in GitHub Actions UI
- **Detailed Logs** - Complete build logs for debugging
- **Downloadable Reports** - Artifact with full build details
- **Timeout Protection** - Automatic timeout after 60 minutes

### Sample Output

```
🚀 Starting Arato build...
✅ Build started successfully
Found experiments to monitor: ["exp123", "exp456"]

🔍 Monitoring build progress...
Monitoring 2 experiment(s)...
🔄 Check 1/120
  ⏳ exp123 still running
  ⏳ exp456 still running
🔄 Check 2/120
  ✅ exp123 completed
  ⏳ exp456 still running
🔄 Check 3/120
  ✅ exp456 completed
🎉 All experiments finished!

✅ Build Success: 2 completed, 0 failed
```

## 🚨 Troubleshooting

### Common Issues

#### ❌ "Invalid JSON format for API keys"
**Problem:** Your `api_keys` parameter is not valid JSON.

**Solution:** Ensure proper JSON formatting:
```yaml
# ✅ Correct
api_keys: '{"openai_api_key": "sk-..."}'

# ❌ Wrong
api_keys: '{openai_api_key: sk-...}'
```

#### ❌ "Build failed with HTTP 401"
**Problem:** Invalid or missing Arato API key.

**Solution:** 
1. Check your `ARATO_API_KEY` secret is set correctly
2. Ensure the key starts with `ar-`
3. Verify the key hasn't expired

#### ❌ "No experiments to monitor"
**Problem:** The build API didn't return any experiment IDs.

**Solution:**
1. Check your experiment IDs are in format `flow_id/experiment_id`
2. Verify the experiments exist in your Arato workspace
3. Check the build API response for errors

#### ⏰ "Builds taking too long"
**Problem:** Experiments are still running after 60 minutes.

**Solution:** This is normal for complex experiments. The workflow will:
- Report current status
- Mark incomplete experiments as "in progress"
- You can check Arato dashboard for final results

### Getting Help

1. **Check the logs** - GitHub Actions provides detailed execution logs
2. **Verify inputs** - Ensure all required parameters are provided correctly
3. **Test manually** - Try your API calls manually using curl or Postman
4. **Check Arato status** - Verify your experiments in the Arato dashboard

## 🔄 Version Management

### Recommended Usage

```yaml
# Use specific version (recommended for production)
uses: AratoAi/arato@v1.0.0

# Use latest stable (for development)  
uses: AratoAi/arato@main
```

### Version History

- **v1.0.0** - Initial release with monitoring and reporting
- **v1.1.0** - Added custom API endpoint support
- **v1.2.0** - Enhanced error handling and timeouts

## 📚 More Examples

Check the [examples/](examples/) directory for complete workflow templates:

- **Basic Usage** - Simple single experiment build
- **Multi-Environment** - Deploy to staging and production
- **Conditional Builds** - Build only changed experiments
- **Direct Action** - Using the action directly in steps
- **Matrix Builds** - Test multiple configurations

## 🤝 Support

- 📖 **Documentation** - Check this README and examples
- 🐛 **Issues** - Report bugs in the Issues tab
- 💬 **Discussions** - Ask questions in Discussions
- 📧 **Contact** - Reach out to the Arato team

---

**Made with ❤️ for AI builders using Arato**
