---
description: Pause an active session (context switch to another task).
---

# /ferret:pause – Pause session

## Arguments: $ARGUMENTS
Session ID, task ID, or empty (if one active session).

## Instructions

1. Find session: if `$ARGUMENTS` empty and one active session → use it.
   If multiple active → list them with elapsed time, ask which to pause.

2. Compute elapsed: `now - started_at` (or `now - resumed_at` if was resumed).
   Add to `elapsed_seconds`.

3. Update JSON: `status → "paused"`, `paused_at → now`.

4. Token snapshot.

5. Output:
```
⏸ Paused: PROJ-123 – Implement auth service  (1h 12min so far)
  /ferret:resume abc12345  to continue
```
