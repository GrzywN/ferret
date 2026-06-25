---
name: session-manager
description: Manages session lifecycle (start, pause, resume, done). Handles elapsed time calculation across multiple pause/resume cycles.
---

# Session Manager

## Elapsed time across pause/resume cycles

```
total_elapsed = sum of all (active_period_end - active_period_start)
```

If session is currently active: add `(now - max(started_at, resumed_at))`.

## Atomic file writes

```bash
TMPFILE="data/sessions/${UUID}.tmp.json"
echo "$JSON" > "$TMPFILE"
mv "$TMPFILE" "data/sessions/${UUID}.json"
```

## State machine

```
created → active → paused → active → ... → completed
                                          → abandoned
```

Never delete sessions. Archive to `data/completed/` on done.
