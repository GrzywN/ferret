---
description: Begin tracking a task. Records start time, token baseline, and your plan. Does not create worktrees.
---

# /ferret:start – Start session tracking

## Arguments: $ARGUMENTS

- `PROJ-123` – task ID from Jira/Linear
- `--estimate 2h` – expected duration (optional but improves /ferret:score)
- `--branch feat/PROJ-123` – note the branch you're working on (optional)
- `--worktree ../path` – note the worktree path (optional, for metadata only)
- `--manual "description"` – track without an issue ID

Worktree setup is the user's responsibility. These flags only record metadata.

## Instructions

### Step 1 – Check for existing session
```bash
ls data/sessions/ 2>/dev/null | grep -l '"task_id":"PROJ-123"' || true
```
If active session found for this task: warn and offer to resume.

### Step 2 – Resolve task info
If Jira/Linear MCP is connected: fetch title and URL for the task ID.
Otherwise: use task ID as title, or the `--manual` description.

### Step 3 – Token baseline
Apply heuristics from CLAUDE.md. Record as `token_snapshots[0]`.

### Step 4 – Parse estimate
"2h" → 7200s  ·  "90min" → 5400s  ·  "1h30m" → 5400s  ·  bare number → hours

### Step 5 – Count concurrent sessions
```bash
CONCURRENT=$(ls data/sessions/*.json 2>/dev/null | wc -l | tr -d ' ')
```
Record as `concurrent_at_start`.

### Step 6 – Ask for plan
"What's your plan for $TASK_TITLE? (1-3 sentences, or press Enter to skip)"
Store in `plan` field.

### Step 7 – Write session JSON
Generate UUID:
```bash
uuidgen 2>/dev/null || cat /proc/sys/kernel/random/uuid 2>/dev/null || date +%s%N | sha256sum | head -c 36
```
Write to `data/sessions/{uuid}.json`. See CLAUDE.md for full schema.

### Step 8 – Output
```
✓ Tracking started
  Task:       PROJ-123 – Implement auth service
  Session ID: abc12345
  Started:    14:32
  Estimate:   2h
  Concurrent: 2 other active sessions

/ferret:status  → see all sessions
/ferret:done abc12345  → when finished
```
