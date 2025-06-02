# Threshold Validation Example

This example demonstrates how to use the threshold validation feature to automatically fail builds when performance metrics don't meet your requirements.

## Basic Usage with Thresholds

```yaml
name: Arato AI Build with Thresholds

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  arato-build:
    runs-on: ubuntu-latest
    steps:
      - name: Build and monitor Arato experiments
        uses: AratoAi/arato-github-actions@v0.0.1
        with:
          experiments: '["flow_123/experiment_456", "flow_123/experiment_789"]'
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

## Threshold Configuration

The `threshold` input accepts a JSON object with the following optional fields:

### `eval_pass_rate` (number)
- **Description**: Minimum pass rate for evaluation validations
- **Range**: 0.0 to 1.0 (percentage as decimal)
- **Validation**: Checks each key in `total_validations_pass_percentage` object
- **Failure condition**: Any validation type has a pass rate below this threshold

### `cost` (number)
- **Description**: Maximum allowed cost per experiment
- **Range**: Any positive number
- **Validation**: Checks the `cost` field in results
- **Failure condition**: Cost exceeds this threshold (skipped if cost is 0)

### `latency` (number)
- **Description**: Maximum allowed time-to-first-token in milliseconds
- **Range**: Any positive number
- **Validation**: Checks the `ttft` field in results
- **Failure condition**: Latency exceeds this threshold

### `tokens` (number)
- **Description**: Maximum allowed token count
- **Range**: Any positive integer
- **Validation**: Checks the `tokens` field in results
- **Failure condition**: Token count exceeds this threshold

### `semantic_similarity` (number)
- **Description**: Minimum required semantic similarity score
- **Range**: -1.0 to 1.0
- **Validation**: Checks the `semantic_similarity` field in results
- **Failure condition**: Similarity score is below this threshold (skipped if value is -1)

## Example: Quality Gate for Production

```yaml
- name: Production Quality Gate
  uses: your-org/arato-github-actions@v1
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

## Example: Development Environment (Relaxed Thresholds)

```yaml
- name: Development Build
  uses: your-org/arato-github-actions@v1
  with:
    experiments: ${{ inputs.dev_experiments }}
    api_keys: ${{ secrets.API_KEYS }}
    arato_api_key: ${{ secrets.ARATO_API_KEY }}
    threshold: |
      {
        "eval_pass_rate": 0.7,
        "latency": 5000
      }
```

## Outputs

When threshold validation is enabled, the action provides additional outputs:

- `threshold_failures`: JSON array of detailed failure messages
- `success`: Will be `false` if any thresholds are violated
- `results_summary`: Includes threshold validation status

## Build Report

The action generates a detailed build report that includes:

1. **Threshold Status**: Whether all thresholds passed
2. **Detailed Failures**: Specific metrics that failed validation
3. **Per-Experiment Results**: Individual experiment performance

Example failure output:
```
❌ Threshold Failures:
- eval_pass_rate: PROMPT_COHERENCE (0.65) below threshold (0.8) in experiment flow_123/experiment_456
- latency: 3500 ms above threshold (3000 ms) in experiment flow_123/experiment_789
```

## Best Practices

1. **Start with relaxed thresholds** and tighten them over time
2. **Use different thresholds** for different environments (dev/staging/prod)
3. **Monitor threshold violations** to understand your model's performance characteristics
4. **Set cost thresholds** to prevent unexpected billing spikes
5. **Use semantic similarity thresholds** to catch significant output quality regressions
