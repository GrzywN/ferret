---
name: token-tracker
description: Estimates token usage per session using the best available method.
---

# Token Tracker

## Detection priority

1. `$ANTHROPIC_TOKENS_USED` (env var from harness/wrapper)
2. Parse `~/.claude/sessions/` latest session: `usage.input_tokens + usage.output_tokens`
3. Manual input at /ferret:done
4. Estimate: `elapsed_minutes × 800` + set `token_estimate: true`

## Snapshot strategy

Take snapshots at: session start, each /ferret:annotate call, session end.
Store deltas per snapshot. Reconstruct total: sum of deltas.
