# Basic Usage Example

Simple workflow for building a single Arato experiment.

```yaml
# .github/workflows/arato-build-basic.yml
name: Basic Arato Build

on:
  workflow_dispatch:
    inputs:
      experiment:
        description: 'Experiment to build (format: flow_id/experiment_id)'
        required: true
        default: 'my-flow/my-experiment'

jobs:
  build:
    name: Build AI Model
    runs-on: ubuntu-latest
    steps:
      - name: Build experiment
        id: arato-build
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ inputs.experiment }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}

      - name: Display build results
        if: always()
        run: |
          echo "Build completed!"
          echo "Success: ${{ steps.arato-build.outputs.success }}"
          echo "Completed: ${{ steps.arato-build.outputs.completed_count }}"
          echo "Failed: ${{ steps.arato-build.outputs.failed_count }}"
```

## Required Secrets

Add these to your repository secrets:

- `ARATO_API_KEY` - Your Arato API key
- `OPENAI_API_KEY` - Your OpenAI API key

## Usage

1. Go to Actions tab in your repository
2. Select "Basic Arato Build"
3. Click "Run workflow"
4. Enter your experiment ID (e.g., `customer-support/sentiment-analysis`)
5. Click "Run workflow"
