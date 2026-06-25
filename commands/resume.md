---
description: Resume a paused session.
---

# /ferret:resume – Resume session

## Arguments: $ARGUMENTS
Session ID, task ID, or empty (if one paused session).

## Instructions

1. Find paused session. If multiple → list, ask.
2. Update JSON: `status → "active"`, `resumed_at → now`, `paused_at → null`.
3. Token snapshot.
4. Output:
```
▶ Resumed: PROJ-123 – Implement auth service
  Elapsed so far: 1h 12min  |  Tokens: ~18,000
```
