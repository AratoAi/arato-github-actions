# CI/CD Pipeline Example

Automatic builds triggered by code changes with smart detection.

```yaml
# .github/workflows/arato          
          if [[ "$staging_success" == "success" && "$prod_success" == "true" ]]; then
            echo "status=✅ Success" >> $GITHUB_OUTPUT
            echo "message=All experiments deployed successfully to production." >> $GITHUB_OUTPUT
          elif [[ "$staging_success" == "success" ]]; then
            echo "status=⚠️ Partial Success" >> $GITHUB_OUTPUT
            echo "message=Experiments tested successfully on staging but not deployed to production." >> $GITHUB_OUTPUT
          else
            echo "status=❌ Failed" >> $GITHUB_OUTPUT
            echo "message=Experiments failed testing on staging." >> $GITHUB_OUTPUT
          fiame: AI Model CI/CD Pipeline

on:
  push:
    branches: [main, develop]
    paths:
      - 'experiments/**'
      - 'models/**'
      - 'data/**'
  pull_request:
    branches: [main]
    paths:
      - 'experiments/**'
      - 'models/**'

env:
  STAGING_API: 'https://staging.arato.ai'
  PROD_API: 'https://api.arato.ai'

jobs:
  detect-changes:
    name: Detect Changed Experiments
    runs-on: ubuntu-latest
    outputs:
      experiments: ${{ steps.detect.outputs.experiments }}
      has_changes: ${{ steps.detect.outputs.has_changes }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2
      
      - name: Detect changed experiments
        id: detect
        run: |
          # Get list of changed files
          changed_files=$(git diff --name-only HEAD~1 HEAD)
          
          # Extract experiment IDs from changed paths
          experiments=()
          
          while IFS= read -r file; do
            if [[ "$file" =~ ^experiments/([^/]+)/([^/]+) ]]; then
              flow_id="${BASH_REMATCH[1]}"
              exp_id="${BASH_REMATCH[2]}"
              experiments+=("$flow_id/$exp_id")
            fi
          done <<< "$changed_files"
          
          # Remove duplicates and convert to JSON
          if [[ ${#experiments[@]} -gt 0 ]]; then
            unique_experiments=($(printf '%s\n' "${experiments[@]}" | sort -u))
            experiments_json=$(printf '%s\n' "${unique_experiments[@]}" | jq -R . | jq -s .)
            echo "experiments=$experiments_json" >> $GITHUB_OUTPUT
            echo "has_changes=true" >> $GITHUB_OUTPUT
            
            echo "🔍 Detected changes in experiments:"
            printf '  - %s\n' "${unique_experiments[@]}"
          else
            echo "experiments=[]" >> $GITHUB_OUTPUT
            echo "has_changes=false" >> $GITHUB_OUTPUT
            echo "ℹ️ No experiment changes detected"
          fi

  test-staging:
    name: Test on Staging
    needs: detect-changes
    if: needs.detect-changes.outputs.has_changes == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Test experiments on staging
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ needs.detect-changes.outputs.experiments }}
          api_base_url: ${{ env.STAGING_API }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.ANTHROPIC_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_API_KEY }}
        }
    secrets:
      ARATO_API_KEY: ${{ secrets.ARATO_STAGING_API_KEY }}

  deploy-production:
    name: Deploy to Production
    needs: [detect-changes, test-staging]
    if: |
      github.ref == 'refs/heads/main' &&
      needs.detect-changes.outputs.has_changes == 'true' &&
      needs.test-staging.result == 'success'
    runs-on: ubuntu-latest
    outputs:
      success: ${{ steps.prod-deploy.outputs.success }}
    steps:
      - name: Deploy to production
        id: prod-deploy
        uses: AratoAi/arato@v1.0.0
        with:
          experiments: ${{ needs.detect-changes.outputs.experiments }}
          api_base_url: ${{ env.PROD_API }}
          api_keys: |
            {
              "openai_api_key": "${{ secrets.PROD_OPENAI_API_KEY }}",
              "anthropic_api_key": "${{ secrets.PROD_ANTHROPIC_API_KEY }}"
            }
          arato_api_key: ${{ secrets.ARATO_PROD_API_KEY }}

  notify:
    name: Send Notifications
    needs: [detect-changes, test-staging, deploy-production]
    if: always() && needs.detect-changes.outputs.has_changes == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Determine status
        id: status
        run: |
          staging_success="${{ needs.test-staging.result }}"
          prod_success="${{ needs.deploy-production.outputs.success }}"
          
          if [[ "$staging_success" == "true" && "$prod_success" == "true" ]]; then
            echo "status=✅ Success" >> $GITHUB_OUTPUT
            echo "message=All experiments deployed successfully to production!" >> $GITHUB_OUTPUT
          elif [[ "$staging_success" == "true" ]]; then
            echo "status=⚠️ Partial Success" >> $GITHUB_OUTPUT
            echo "message=Experiments tested successfully on staging but not deployed to production." >> $GITHUB_OUTPUT
          else
            echo "status=❌ Failed" >> $GITHUB_OUTPUT
            echo "message=Experiments failed testing on staging." >> $GITHUB_OUTPUT
          fi

      - name: Post to Slack
        if: env.SLACK_WEBHOOK_URL != ''
        run: |
          curl -X POST -H 'Content-type: application/json' \
            --data '{
              "text": "AI Model Build: ${{ steps.status.outputs.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "${{ steps.status.outputs.message }}\n\n*Experiments:* ${{ needs.detect-changes.outputs.experiments }}\n*Branch:* `${{ github.ref_name }}`\n*Commit:* <${{ github.event.head_commit.url }}|${{ github.event.head_commit.id }}>"
                  }
                }
              ]
            }' \
            ${{ env.SLACK_WEBHOOK_URL }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Required Secrets

### Staging Environment
- `ARATO_STAGING_API_KEY` - Staging Arato API key
- `OPENAI_API_KEY` - OpenAI API key for staging

### Production Environment  
- `ARATO_PROD_API_KEY` - Production Arato API key
- `PROD_OPENAI_API_KEY` - Production OpenAI API key
- `PROD_ANTHROPIC_API_KEY` - Production Anthropic API key

### Optional
- `SLACK_WEBHOOK_URL` - Slack webhook for notifications

## Directory Structure

Organize your experiments like this:

```
experiments/
├── customer-support/
│   ├── sentiment-analysis/
│   │   ├── config.yaml
│   │   └── prompts.txt
│   └── ticket-routing/
│       ├── config.yaml
│       └── prompts.txt
└── marketing/
    └── email-classification/
        ├── config.yaml
        └── prompts.txt
```

## Features

- **Smart Detection** - Only builds changed experiments
- **Multi-Environment** - Test on staging, deploy to production
- **Safety Checks** - Production deploys only after staging success
- **Notifications** - Slack integration for build status
- **Branch Protection** - Production deploys only from main branch

## Customization

### Change Detection Logic

Modify the `detect-changes` job to match your file structure:

```bash
# For different directory structures
if [[ "$file" =~ ^models/([^/]+) ]]; then
  experiments+=("$BASH_REMATCH[1]")
fi

# For YAML config files
if [[ "$file" =~ ^config/([^/]+)\.yaml$ ]]; then
  experiments+=("$BASH_REMATCH[1]")
fi
```

### Environment Variables

Add environment-specific configuration:

```yaml
env:
  STAGING_CONFIG: |
    {
      "max_tokens": 1000,
      "temperature": 0.7
    }
  PROD_CONFIG: |
    {
      "max_tokens": 2000,
      "temperature": 0.5
    }
```
