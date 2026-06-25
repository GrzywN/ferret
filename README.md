# 🐾 Ferret

> Claude Code plugin for local-first time tracking in AI-assisted development.
> Parallel task analysis · Session lifecycle · Token metrics · Git-native

## Install

### Jednorazowa sesja (najszybciej)
```bash
claude --plugin-dir /absolutna/sciezka/do/ferret
```

### Permanentna (wszystkie projekty)

```bash
# Temporary (dev/test):
claude --plugin-dir ./agent-tracker

# Permanent (your user profile):
claude plugin install ./agent-tracker --scope user

# Per-project (committed to repo):
claude plugin install ./agent-tracker --scope project
```

Then use `/ferret:` commands in any Claude Code session.

## Commands

| Command | What it does |
|---------|-------------|
| `/ferret:plan` | Analyse task dependencies → find what can run in parallel |
| `/ferret:start PROJ-123` | Start tracking a task |
| `/ferret:pause` / `/ferret:resume` | Context switch |
| `/ferret:done` | Complete session |
| `/ferret:status` | Show active sessions |
| `/ferret:annotate` | Append `Claude-Time:` trailer to last commit |
| `/ferret:brag` | *(optional)* Generate XYZ eval / case-study / blog |
| `/ferret:score` | Parallelism · velocity · token efficiency report |

## /ferret:plan is read-only

It only **analyses and reports**. It does not create worktrees, branches, or sessions.
Orchestration is yours – use whatever SDLC tooling your team uses.

```
/ferret:plan --source linear

📋 PARALLELIZATION ANALYSIS
✅ CAN RUN IN PARALLEL: PROJ-123, PROJ-127, PROJ-131
⛔ MUST BE SEQUENTIAL: PROJ-124 [needs: PROJ-123]
Critical path: PROJ-123 → PROJ-124 (5h minimum)
Speedup opportunity: 2× vs sequential
```

## Tracking sessions

Sessions are plain JSON files in `data/sessions/`. Nothing leaves your machine.

```bash
/ferret:start PROJ-123 --estimate 2h
# ... work ...
/ferret:annotate          # add Claude-Time: to last commit
/ferret:done              # complete, write checkpoint
```

## Git integration

```
feat: implement auth service

Claude-Time: elapsed=2h15m tokens=45231 session=abc12345 concurrent=2
```

Optional hook for automatic trailer on every commit:
```bash
cp hooks/prepare-commit-msg /your-project/.git/hooks/
chmod +x /your-project/.git/hooks/prepare-commit-msg
export FERRET_DIR=/path/to/agent-tracker
```

## Brag docs (optional)

```bash
/ferret:brag --style case-study --period "last-30-days"
/ferret:brag --style blog --tasks PROJ-123
```

Uses **Google XYZ Framework**: "Accomplished [X] as measured by [Y], by doing [Z]."
Extended for AI: token cost and concurrency as Y metrics.

## Scoring

```
overall = 0.35 × parallelism + 0.30 × velocity + 0.25 × token_efficiency + 0.10 × quality
```

```bash
/ferret:score --compare   # current vs previous period
```

---

*Open source · Local-first · No external services required*
*Works with any LLM harness, not just Claude Code*
