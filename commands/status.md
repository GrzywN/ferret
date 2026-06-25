---
description: Show all active and paused sessions with elapsed time and token burn.
---

# /ferret:status – Session overview

## Arguments: $ARGUMENTS
- `--all` include recently completed (last 7 days)
- `--json` raw output

## Instructions

Read all files from `data/sessions/`. For each, compute live elapsed time.

Display:
```
⏱  ACTIVE SESSIONS (2)
══════════════════════════════════════

▶ PROJ-123 – Implement auth service
  Session: abc12345  |  Branch: feat/PROJ-123 (if set)
  Started: 14:32  (1h 23min ago)
  Elapsed: 1h 23min  |  Estimate: 2h  ████████░░  (69%)
  Tokens:  ~18,000 (est)
  Plan:    Implement JWT auth with refresh tokens

⏸ PROJ-127 – Update API docs  [PAUSED 2h 10min ago]
  Elapsed: 0h 45min

──────────────────────────────────────
Concurrent: 2  |  Token burn: ~215/min
```

Warnings:
- Session exceeds estimate by >50% → "⚠ Over estimate"
- No active sessions → "No active sessions. Run /ferret:start TASK-ID to begin."
