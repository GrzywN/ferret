# Ferret – Plugin Orchestrator

> Ferret out how long things actually take.
> Plugin namespace: `ferret`  →  `/ferret:plan` `/ferret:start` `/ferret:pause` `/ferret:resume`
>                            `/ferret:done` `/ferret:status` `/ferret:annotate` `/ferret:brag` `/ferret:score`

---

## Commands overview

| Command | Purpose | Creates files? |
|---------|---------|---------------|
| `/ferret:plan` | Parallelization analysis – read-only report | No |
| `/ferret:start` | Begin session tracking | `data/sessions/*.json` |
| `/ferret:pause` | Pause active session | updates session JSON |
| `/ferret:resume` | Resume paused session | updates session JSON |
| `/ferret:done` | Complete session | moves to `data/completed/` |
| `/ferret:status` | Show active sessions | No |
| `/ferret:annotate` | Annotate last git commit + append NDJSON | `.ferret.ndjson` |
| `/ferret:brag` | *(optional)* Generate XYZ doc | `output/` |
| `/ferret:score` | Analytics & performance score | `output/reports/` |

---

## Session JSON format

`data/sessions/{uuid}.json`:
```json
{
  "id": "uuid",
  "task_id": "PROJ-123",
  "task_title": "Implement auth service",
  "task_url": "https://linear.app/...",
  "source": "linear|jira|github|manual",
  "worktree": null,
  "branch": null,
  "status": "active|paused|completed",
  "started_at": "ISO8601",
  "paused_at": null,
  "resumed_at": null,
  "completed_at": null,
  "elapsed_seconds": 0,
  "estimate_seconds": 7200,
  "concurrent_at_start": 0,
  "plan": "Free-text plan",
  "commits": [],
  "token_snapshots": [{ "at": "start", "tokens": 0, "ts": "ISO8601" }],
  "token_total": 0,
  "token_estimate": true,
  "context_notes": "",
  "tags": []
}
```

Completed sessions move to `data/completed/` and gain:
`actual_elapsed_seconds`, `outcome` (shipped|needs-revision|abandoned), `revision_count`, `summary`

---

## NDJSON log format (.ferret.ndjson)

One JSON per line, append-only (`>>`), never overwrite:
```json
{"type":"checkpoint","session":"abc123","task":"PROJ-123","commit":"def456ab","elapsed_delta":2700,"tokens_delta":15000,"ts":"ISO8601","concurrent":2}
{"type":"done","session":"abc123","task":"PROJ-123","commit":"def456ab","elapsed_total":8100,"tokens_total":45231,"outcome":"shipped","revision_count":0,"ts":"ISO8601"}
```

---

## Token tracking heuristics (priority order)

1. `$ANTHROPIC_TOKENS_USED` env var (if set by harness)
2. `~/.claude/sessions/` latest session usage fields
3. Manual input at `/ferret:done`
4. Estimation fallback: `elapsed_minutes × 800`

Always set `"token_estimate": true` when using estimation.

---

## Google XYZ Framework (for /ferret:brag)

> "Accomplished [X] as measured by [Y], by doing [Z]."

AI-era Y metrics: token cost, concurrency factor, under/over estimate
AI-era Z: "orchestrated via Claude Code", "ran 3 parallel worktrees"
