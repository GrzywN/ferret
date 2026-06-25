---
description: Annotate the last git commit with a Claude-Time trailer and write an NDJSON checkpoint. Separate from brag doc generation.
---

# /ferret:annotate – Git annotation

## Arguments: $ARGUMENTS
- `--session abc12345` – specific session (default: detect from active)
- `--task PROJ-123`
- `--squash` – reconstruct totals from all NDJSON checkpoints for this session
- `--notes` – use git notes instead of amending (safe for already-pushed commits)
- `--dry-run` – show what would be written, change nothing

## Instructions

### Step 1 – Detect session
From `$ARGUMENTS` or find the single active session. If multiple active and no flag: list and ask.

### Step 2 – Compute values

**Normal mode:** use session's current `elapsed_seconds` and `token_total`.

**`--squash` mode:** reconstruct from NDJSON:
```bash
grep '"session":"abc12345"' .ferret.ndjson 2>/dev/null | \
  python3 -c "
import sys, json
lines = [json.loads(l) for l in sys.stdin if l.strip()]
checkpoints = [l for l in lines if l.get('type') == 'checkpoint']
print(json.dumps({
  'elapsed': sum(c.get('elapsed_delta',0) for c in checkpoints),
  'tokens': sum(c.get('tokens_delta',0) for c in checkpoints),
  'concurrent': max((c.get('concurrent',0) for c in checkpoints), default=0)
}))" 2>/dev/null || echo '{"elapsed":0,"tokens":0,"concurrent":0}'
```

### Step 3 – Format trailer

```
Claude-Time: elapsed=2h15m tokens=45231 session=abc12345 concurrent=2
```

Rules:
- elapsed: "Xh Ymin" or "Ymin" if < 60min
- tokens: exact, or append `~` if estimated
- session: first 8 chars of UUID

### Step 4 – Apply

**Default (amend):**
```bash
git commit --amend --no-edit --trailer "Claude-Time: elapsed=... tokens=... session=... concurrent=..."
```
Warn if commit is already pushed to a remote.

**`--notes` mode (safe, non-destructive):**
```bash
git notes add -m "Claude-Time: elapsed=... tokens=... session=... concurrent=..." HEAD
```

### Step 5 – Write NDJSON checkpoint
Append to `.ferret.ndjson`:
```json
{"type":"checkpoint","session":"abc12345","task":"PROJ-123","commit":"abcdef12","elapsed_delta":2700,"tokens_delta":15000,"ts":"ISO8601","concurrent":2}
```

### Step 6 – Output
```
✓ Annotated commit abcdef12
  Claude-Time: elapsed=2h15m tokens=45,231 session=abc12345 concurrent=2
  Method: commit --amend  (or: git notes)
  Checkpoint: appended to .ferret.ndjson
```
