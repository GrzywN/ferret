---
description: Analyse task dependencies and identify parallelization opportunities. Read-only report – does not create worktrees or sessions.
---

# /ferret:plan – Parallelization analysis

**This command only analyses and reports. It creates nothing.**
Worktree setup, branch creation, and task assignment are the user's responsibility
and depend on their SDLC, tooling, and team processes.

## What it does
1. Fetches tasks from Jira/Linear (MCP) or accepts a manual list
2. Builds a dependency graph
3. Identifies which tasks can run in parallel and which cannot
4. Reports potential file conflicts between parallel tasks
5. Outputs a recommended execution order

## Arguments: $ARGUMENTS

Parse from `$ARGUMENTS`:
- `--source linear|jira|github|manual` (default: ask)
- `--query "sprint:current status:todo"` (passed to MCP search)
- `--json` (output as JSON instead of prose)

## Instructions

### Step 1 – Fetch tasks

**Linear:** use Linear MCP tool `linear_list_issues` or similar.
Extract per issue: id, title, estimate (points/hours), status, blocking[], blockedBy[], parentId, url.

**Jira:** use Jira MCP `searchJiraIssuesUsingJql`.
JQL examples: `sprint in openSprints() AND status = "To Do" AND assignee = currentUser()`
Extract: key, summary, timeestimate, issuelinks (Blocks / is blocked by).

**Manual:** if no MCP or `--source manual`, ask:
"Paste tasks (one per line): `ID: Title (estimate)` — add `[after: ID]` for deps"

### Step 2 – Build dependency graph

```
graph = { task_id: { title, estimate, depends_on: [], blocks: [] } }
```

Infer implicit dependencies when explicit data missing:
- "integrate X" after "implement X" in same sprint → likely depends
- Same parent epic + sequential numbering → possible order constraint
- "blocked" / "after" / "depends" in title → strong signal

### Step 3 – Classify tasks

For each pair of tasks (A, B), check:

| Condition | Classification |
|-----------|---------------|
| B.depends_on includes A | Sequential: B after A |
| A.depends_on includes B | Sequential: A after B |
| Both in same file paths (if inferable) | Warn: potential conflict |
| No relationship found | Parallel-safe |

### Step 4 – Output report

```
📋 PARALLELIZATION ANALYSIS
────────────────────────────────────────────────

✅ CAN RUN IN PARALLEL:
  • PROJ-123  Implement auth service        est: 2h   [no dependencies]
  • PROJ-127  Update API documentation      est: 1h   [no dependencies]
  • PROJ-131  Add Redis caching             est: 3h   [no dependencies]

  ⚠ Potential conflict: PROJ-123 and PROJ-131 may both touch src/config/
    → Review before running in parallel

⛔ MUST BE SEQUENTIAL:
  • PROJ-124  Frontend auth integration     est: 3h   [needs: PROJ-123]
  • PROJ-128  Update docs with new API      est: 1h   [needs: PROJ-127]

📊 RECOMMENDED GROUPS:

  Phase 1 (parallel): PROJ-123, PROJ-127, PROJ-131
    → Min time if parallel: 3h  (vs 6h sequential = 2× speedup)

  Phase 2 (after Phase 1): PROJ-124, PROJ-128
    → Min time if parallel: 3h

  Critical path: PROJ-123 → PROJ-124 (total: 5h minimum)

────────────────────────────────────────────────
This is an analysis only. Worktree setup and task assignment
are yours to manage with your tooling of choice.

To start tracking once you're set up:
  /ferret:start PROJ-123 --estimate 2h
  /ferret:start PROJ-127 --estimate 1h    ← run in separate worktree/terminal
```

If `--json`: output a JSON object with `parallel_groups`, `sequential_order`, `conflicts`, `critical_path`.
