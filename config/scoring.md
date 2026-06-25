# Scoring Formulas

## Parallelism (0-100)
```
parallel_sessions = count(sessions where concurrent_at_start >= 1)
parallelism = (parallel_sessions / total_sessions) * 100
if avg_concurrent > 2: parallelism = min(parallelism * 1.15, 100)
```

## Velocity (0-100) — only sessions with estimates
```
ratio = estimate_seconds / actual_elapsed_seconds
ratio_capped = min(ratio, 2.0)
velocity = mean(ratio_capped_all) * 50
```

## Token efficiency (0-100) — needs ≥3 prior sessions as baseline
```
tok_per_hour = token_total / (elapsed_seconds / 3600)
baseline = rolling_avg(tok_per_hour, last_10_sessions)
efficiency = min((baseline / tok_per_hour) * 100, 200)
```
>100 = more efficient than baseline, <100 = less efficient

## Quality (0-100)
```
quality = mean(max(0, 100 - revision_count * 20)) per session
```

## Overall
```
overall = 0.35 * parallelism + 0.30 * velocity + 0.25 * efficiency + 0.10 * quality
```
Thresholds: ≥80 Excellent · 60-79 Good · 40-59 Needs work · <40 Review
