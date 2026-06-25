---
description: Calculate parallelism, velocity, token efficiency, and quality scores from completed sessions.
---

# /ferret:score – Performance analytics

## Arguments: $ARGUMENTS
- `--period "March 2026"` / `"Q1 2026"` / `"last-30-days"` / `"all"`
- `--compare` – current vs previous period
- `--breakdown` – per-task detail
- `--json` – raw output

## Instructions

### Step 1 – Load data
Read all `data/completed/*.json` for period + `.ferret.ndjson`.
Compute baseline: rolling average of all prior periods.

### Step 2 – Scores (see config/scoring.md for formulas)

**Parallelism (0-100):** % of sessions where `concurrent_at_start >= 1`
Bonus ×1.15 if avg concurrent > 2 (capped at 100)

**Velocity (0-100):** sessions with estimates only
`mean(min(estimate/actual, 2)) × 50`

**Token efficiency (0-100):** relative to your own baseline
`min((baseline_tok_per_hour / current_tok_per_hour) × 100, 200)`
N/A if < 3 completed sessions.

**Quality (0-100):** `mean(max(0, 100 - revision_count × 20))`

**Overall:** `0.35 × P + 0.30 × V + 0.25 × E + 0.10 × Q`

Thresholds: ≥80 🟢 · 60-79 🟡 · 40-59 🟠 · <40 🔴

### Step 3 – Render report

```
📊 PERFORMANCE SCORE – March 2026
══════════════════════════════════════════════

Overall:     74 / 100  🟡 Good

  Parallelism:  82 / 100  ████████░░  avg 2.1 concurrent
  Velocity:     78 / 100  ███████░░░  12% over estimates avg
  Token eff.:   68 / 100  ██████░░░░  310 tok/min (baseline 265)
  Quality:      85 / 100  ████████░░  0.3 revisions avg

12 tasks · 47.5h · ~890k tokens

OPTIMISATION SUGGESTIONS
  → 4 tasks had no estimate – add estimates to track velocity
  → PROJ-135 ran solo – could it have paralleled PROJ-137?
  → Token efficiency ↓15% vs Feb – check context window growth
  → 2 tasks >50% over estimate – consider smaller scopes
```

Save to `output/reports/YYYY-MM-score.md`.
If `--compare`: show delta vs previous period.
