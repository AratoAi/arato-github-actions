# Multiple Experiments Example

Build multiple experiments in a single workflow run.

```yaml
# .github/workflows/arato-build-multiple.yml
name: Build Multiple Experiments

on:
  workflow_dispatch:
    inputs:
      experiments:
        description: 'Experiments to build (JSON array)'
        required: true
        default: '["flow1/exp1", "flow1/exp2", "flow2/exp1"]'

jobs:
  build:
    name: Build AI Models
    runs-on: ubuntu-latest
    steps:
      - name: Build experiments
        id: arato-build
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ inputs.experiments }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}",
              "together_api_key": "${{ secrets.TOGETHER_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}

      - name: Parse results
        if: always()
        run: |
          echo "📊 Build Summary"
          echo "==============="
          
          total_experiments=$(echo '${{ steps.arato-build.outputs.results_summary }}' | jq '.total_count')
          completed=$(echo '${{ steps.arato-build.outputs.results_summary }}' | jq '.completed_count')
          failed=$(echo '${{ steps.arato-build.outputs.results_summary }}' | jq '.failed_count')
          
          echo "Total Experiments: $total_experiments"
          echo "✅ Completed: $completed"
          echo "❌ Failed: $failed"
          
          success_rate=$(( completed * 100 / total_experiments ))
          echo "📈 Success Rate: ${success_rate}%"
          
          if [[ "${{ steps.arato-build.outputs.success }}" == "true" ]]; then
            echo "🎉 All experiments completed successfully!"
          else
            echo "⚠️ Some experiments failed. Check the logs for details."
          fi
```

## Example Experiment Arrays

### Simple List
```json
["customer-support/sentiment", "customer-support/classification"]
```

### Cross-Team Experiments
```json
[
  "marketing/email-classification",
  "support/ticket-routing",
  "sales/lead-scoring"
]
```

### Development vs Production
```json
[
  "dev/model-v1",
  "dev/model-v2",
  "prod/current-model"
]
```

## Required Secrets

- `ARATO_API_KEY` - Your Arato API key
- `OPENAI_API_KEY` - OpenAI API key
- `ANTHROPIC_API_KEY` - Anthropic API key
- `TOGETHER_API_KEY` - Together AI API key

## Tips

1. **Start Small** - Test with 1-2 experiments first
2. **Monitor Resources** - Large batches may take longer
3. **Check Dependencies** - Ensure all experiments are ready to build
4. **Use Parallel Builds** - The action monitors all experiments simultaneously
