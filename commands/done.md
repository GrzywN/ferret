---
description: Complete a session. Records final time, token count, and outcome. Moves session to completed/.
---

# /ferret:done – Complete session

## Arguments: $ARGUMENTS
Session ID / task ID, plus optional:
- `--summary "what was delivered"` (skip interactive prompt)
- `--outcome shipped|needs-revision|abandoned`

## Instructions

### Step 1 – Find session
As with /ferret:pause.

### Step 2 – Final elapsed
`actual_elapsed = stored elapsed_seconds + (now - last active start)`

### Step 3 – Final token count
Check env var / Claude session file. If unavailable, ask:
"Approximately how many tokens? (check DevTools Network, or Enter to estimate)"
If skipped: use `elapsed_minutes × 800`, set `token_estimate: true`.

### Step 4 – Gather outcome (if not in $ARGUMENTS)
Ask:
- "What was delivered? (1-2 sentences)"  → `summary`
- "Outcome: [1] shipped  [2] needs-revision  [3] abandoned"  → default: 1
- "Revision count after calling done? (default 0)"

### Step 5 – Write NDJSON checkpoint
Append to `.ferret.ndjson`:
```json
{"type":"done","session":"ID","task":"PROJ-123","commit":"$(git rev-parse --short HEAD 2>/dev/null || echo none)","elapsed_total":8100,"tokens_total":45231,"outcome":"shipped","revision_count":0,"ts":"ISO8601","concurrent":N}
```

### Step 6 – Move session file
```bash
mv data/sessions/{id}.json data/completed/{id}.json
```
Add final fields to JSON before moving.

### Step 7 – Offer annotation
"Annotate last commit with Claude-Time trailer? (y/N)"
If yes: run /ferret:annotate logic inline for this session.

### Step 8 – Output
```
✅ Done: PROJ-123 – Implement auth service
  Elapsed: 2h 15min  (estimate: 2h — 12% over)
  Tokens:  ~45,231
  Outcome: shipped

/ferret:score  → update metrics
/ferret:brag --tasks PROJ-123  → generate XYZ doc (optional)
```
