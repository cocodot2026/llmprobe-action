# llmprobe-action

**A GitHub Action that fails your CI when your AI API relay is degraded or silently
model-downgraded — automated verification with the open-source
[cocodot-llmprobe](https://github.com/cocodot2026/cocodot-llmprobe).**

If your app depends on a relay serving the *real* model, don't find out it got
swapped for a cheap one when your users do. Run this on a schedule (or on deploy)
and get alerted the moment the score drops.

## Usage

```yaml
name: relay-downgrade-check
on:
  schedule:
    - cron: "0 */6 * * *"   # every 6 hours
  workflow_dispatch:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: cocodot2026/llmprobe-action@v1
        with:
          base-url: https://your-relay/v1
          api-key: ${{ secrets.RELAY_API_KEY }}
          model: your-model-id
          min-score: "70"          # fail below this
```

## Inputs

| input | required | default | description |
|---|---|---|---|
| `base-url` | ✅ | — | OpenAI-compatible base URL of the relay |
| `api-key` | ✅ | — | relay key — **pass a secret**, never hardcode |
| `model` | ✅ | — | model id to verify |
| `min-score` | | `70` | fail the job below this 0–100 score |
| `judge-base-url` / `judge-api-key` | | — | optional independent judge endpoint for the capability probe |

## What it does
1. Installs deps and fetches `llmprobe.py` from the LLMprobe repo.
2. Runs the 6 probes (identity, capability, latency, context, rate-limit, consistency).
3. Parses the `NN/100` score and **fails the job** if it's below `min-score`.

> The score parser matches LLMprobe's `NN/100` output. If you pin a customized
> LLMprobe, adjust the parse step accordingly.

## Where it fits
Part of a small honest toolkit for running AI from China:
[cocodot-llmprobe](https://github.com/cocodot2026/cocodot-llmprobe) (the verifier) ·
[relay-doctor](https://github.com/cocodot2026/relay-doctor) (quick health check) ·
[ai-coding-from-china](https://github.com/cocodot2026/ai-coding-from-china) (the full skill).

The author builds [cocodot](https://cocodot.co), a relay — disclosed; this Action
works against **any** endpoint. MIT.
