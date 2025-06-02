# Arato Build GitHub Action

A GitHub Action for triggering and monitoring Arato AI builds

## 🚀 Quick Start

### 1. Add to Your Workflow

Create `.github/workflows/arato-build.yml` in your repository:

```yaml
name: Arato Build

on:
  workflow_dispatch:
    inputs:
      experiments:
        description: 'Experiments to build (for example my-notebook-id')
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Arato Build
        uses: AratoAi/arato-github-actions@v0.0.1
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
In this example we're using OpenAI and Anthropic but you can use whatever vendor you want

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `ARATO_API_KEY` | Your Arato API key | `ar-1234567890abcdef...` |
| `OPENAI_API_KEY` | OpenAI API key (if used) | `sk-1234567890abcdef...` |
| `ANTHROPIC_API_KEY` | Anthropic API key (if used) | `ant-1234567890abcdef...` |

### 3. Run Your Build

1. Go to **Actions** tab in your repository
2. Select "Arato Build" workflow
3. Click "Run workflow"
4. Enter your experiment ID (e.g., `my-notebook/my-experiment` or `my-notebook`)
5. Click "Run workflow"

## 📋 Configuration

### Inputs

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `experiments` | ✅ Yes | - | Single experiment string or an array of experiments |
| `api_keys` | ✅ Yes | - | JSON object with your AI model API keys |
| `api_base_url` | ❌ No | `https://api.arato.ai` | Arato API endpoint |
| `arato_api_key` | ✅ Yes | - | Your Arato API key (starts with "ar-") |
| `threshold` | ❌ No | - | JSON object with performance thresholds for automatic validation |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | boolean | Whether all builds completed successfully and passed thresholds |
| `completed_count` | number | Number of successful experiments |
| `failed_count` | number | Number of failed experiments |
| `results_summary` | JSON | Detailed build results including threshold validation |
| `threshold_failures` | JSON | Array of threshold validation failures (if any) |

## 💡 Usage Examples

### Single Experiment

```yaml
with:
  experiments: ['customer-support']
  api_keys: '{"openai_api_key": "${{ secrets.OPENAI_API_KEY }}"}'
```

### Multiple Experiments

```yaml
with:
  experiments: '["exp1", "exp2", "exp3"]'
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
      - name: Arato Build
        id: arato-build
        uses: AratoAi/arato-github-actions@v0.0.1
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

### With Performance Thresholds

```yaml
with:
  experiments: '["quality-check", "performance-test"]'
  api_keys: |
    {
      "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
      "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"
    }
  arato_api_key: ${{ secrets.ARATO_API_KEY }}
  threshold: |
    {
      "eval_pass_rate": 0.8,
      "cost": 0.01,
      "latency": 3000,
      "tokens": 100,
      "semantic_similarity": 0.7
    }
```

### Quality Gate for Production

```yaml
- name: Production Quality Gate
  uses: AratoAi/arato-github-actions@v0.0.1
  with:
    experiments: ${{ inputs.production_experiments }}
    api_keys: ${{ secrets.API_KEYS }}
    arato_api_key: ${{ secrets.ARATO_API_KEY }}
    threshold: |
      {
        "eval_pass_rate": 0.95,
        "cost": 0.005,
        "latency": 2000,
        "semantic_similarity": 0.85
      }
```

## 🎯 Performance Thresholds

The action supports automatic validation of experiment results against configurable performance thresholds. When thresholds are provided, builds will fail if any metrics don't meet your requirements.

### Supported Thresholds

| Threshold | Description | Failure Condition |
|-----------|-------------|-------------------|
| `eval_pass_rate` | Minimum evaluation pass rate (0.0-1.0) | Any validation type below threshold |
| `cost` | Maximum cost per experiment | Cost exceeds threshold (skipped if 0) |
| `latency` | Maximum time-to-first-token (ms) | Latency exceeds threshold |
| `tokens` | Maximum token count | Token count exceeds threshold |
| `semantic_similarity` | Minimum similarity score (-1.0-1.0) | Similarity below threshold (skipped if -1) |

### Example Threshold Configuration

```yaml
threshold: |
  {
    "eval_pass_rate": 0.8,
    "cost": 0.01,
    "latency": 3000,
    "tokens": 100,
    "semantic_similarity": 0.7
  }
```

For detailed examples and best practices, see [examples/threshold-validation.md](examples/threshold-validation.md).

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
1. Check your experiment IDs are in format `notebook-id/experiment-id` or `notebook-id`
2. Verify the experiments exist in your Arato workspace
3. Check the build API response for errors

#### ❌ "Invalid JSON format for threshold"
**Problem:** Your `threshold` parameter is not valid JSON.

**Solution:** Ensure proper JSON formatting:
```yaml
# ✅ Correct
threshold: '{"eval_pass_rate": 0.8, "cost": 0.01}'

# ❌ Wrong
threshold: '{eval_pass_rate: 0.8, cost: 0.01}'
```

#### ❌ Build fails due to threshold violations
**Problem:** One or more performance metrics don't meet your thresholds.

**Solution:**
1. Check the detailed failure report in the build summary
2. Review individual experiment results
3. Adjust thresholds if they're too strict for your use case
4. Investigate performance issues in failing experiments


### Getting Help

1. **Check the logs** - GitHub Actions provides detailed execution logs
2. **Verify inputs** - Ensure all required parameters are provided correctly
3. **Check Arato status** - Verify your experiments in the Arato UI

## 📚 More Examples

Check the [examples/](examples/) directory for complete workflow templates:

- [Basic Usage](examples/basic-usage.md) - Simple single experiment builds
- [Multiple Experiments](examples/multiple-experiments.md) - Building multiple experiments in parallel  
- [Threshold Validation](examples/threshold-validation.md) - Performance thresholds and quality gates

## 🤝 Support

- 📖 **Documentation** - Check this README and examples
- 🐛 **Issues** - Report bugs in the Issues tab
- 📧 **Contact** - Reach out to the Arato team

---

**Made with ❤️ for AI builders using Arato**
