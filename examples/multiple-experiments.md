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
        uses: AratoAi/arato-github-actions@v0.0.1
        with:
          experiments: ${{ inputs.experiments }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"              
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

## Required Secrets

- `ARATO_API_KEY` - Your Arato API key

