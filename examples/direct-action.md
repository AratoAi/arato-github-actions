# Direct Action Usage Example

This example shows how to use the Arato Build Action directly as a step in your workflow.

```yaml
# .github/workflows/arato-action-example.yml
name: Arato Direct Action Example

on:
  workflow_dispatch:
    inputs:
      experiment:
        description: 'Experiment to build (format: flow_id/experiment_id)'
        required: true
        default: 'customer-support/sentiment-analysis'
      model_provider:
        description: 'AI model provider to use'
        type: choice
        options:
          - openai
          - anthropic
          - both
        default: 'openai'

jobs:
  build-with-action:
    name: Build using Direct Action
    runs-on: ubuntu-latest
    steps:
      - name: Build Arato Experiment
        id: arato_build
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ inputs.experiment }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}

      - name: Process Results
        run: |
          echo "🔍 Build Results:"
          echo "Success: ${{ steps.arato_build.outputs.success }}"
          echo "Completed: ${{ steps.arato_build.outputs.completed_count }}"
          echo "Failed: ${{ steps.arato_build.outputs.failed_count }}"
          echo "Details: ${{ steps.arato_build.outputs.results_summary }}"
          
          if [[ "${{ steps.arato_build.outputs.success }}" == "true" ]]; then
            echo "🎉 Build completed successfully!"
            echo "Ready for deployment or further testing."
          else
            echo "❌ Build failed. Check the logs above for details."
            exit 1
          fi

      - name: Save Results
        if: always()
        run: |
          mkdir -p build-results
          echo '${{ steps.arato_build.outputs.results_summary }}' > build-results/summary.json
          
      - name: Upload Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: arato-build-results
          path: build-results/

  conditional-build:
    name: Conditional Build Example
    runs-on: ubuntu-latest
    if: inputs.model_provider == 'both'
    steps:
      - name: Build with Multiple Providers
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: '["${{ inputs.experiment }}", "demo-flow/test-experiment"]'
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}",
              "together_api_key": "${{ secrets.TOGETHER_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
```

## Key Features of the Arato Action

### Action Advantages:
- **Simple syntax** - Just use `uses:` like any other action
- **Flexible integration** - Can be combined with other steps in the same job
- **Immediate outputs** - Process results directly in subsequent steps
- **Direct API key input** - Pass API key directly as input parameter
- **Real-time monitoring** - Built-in progress tracking and reporting

### Basic Usage Pattern:
```yaml
# Arato action as a workflow step
- name: Build experiment
  id: arato-build
  uses: AratoAi/arato@v1.0.0
  with:
    experiments: 'my-experiment'
    arato_api_key: ${{ secrets.ARATO_API_KEY }}
    api_keys: '{"openai_api_key": "${{ secrets.OPENAI_API_KEY }}"}'

- name: Use results
  run: |
    echo "Success: ${{ steps.arato-build.outputs.success }}"
```

## Required Secrets

Add these to your repository secrets:

- `ARATO_API_KEY` - Your Arato API key
- `OPENAI_API_KEY` - Your OpenAI API key (if using OpenAI)
- `ANTHROPIC_API_KEY` - Your Anthropic API key (if using Claude)
- `TOGETHER_API_KEY` - Your Together AI API key (if using Together)

## Usage Tips

1. **Start Simple** - Use single experiments first to test your setup
2. **Scale Up** - Use JSON arrays for multiple experiments
3. **Monitor Progress** - The action provides real-time monitoring
4. **Handle Errors** - Always check the `success` output
5. **Save Results** - Use outputs to make decisions in subsequent steps
