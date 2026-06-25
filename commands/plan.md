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
- `--apply-labels` (opt-in write: prefix tracker issue titles with their numeric label — see Step 5)
- `--dry-run` (with `--apply-labels`: print before/after titles, write nothing)

**Default = read-only.** Without `--apply-labels`, this command writes nothing —
no files, no sessions, no tracker edits. The numeric plan is advice only.

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

### Step 4 – Assign numeric labels & output report

Hand the graph to the `concurrency-planner` agent for label assignment. The scheme:

- **`N`** – sequential series. `1` finishes before `2` starts. `series = 1 + max(series of dependencies)`.
- **`N.M`** – parallel track inside series `N`. `1.1` and `1.2` have no dependency between them → run simultaneously.
- **`N.M.K`** – subtask split, **only** when an issue has dedicated subtasks needing separate assignment. Normally absent.

Read at a glance: same first number = same wave; different second number = parallel.

```
📋 ORCHESTRATION PLAN
────────────────────────────────────────────────

SERIES 1  (start now, in parallel):
  1.1  PROJ-123  Implement auth service        est: 2h   [no deps]
  1.2  PROJ-127  Update API documentation      est: 1h   [no deps]
  1.3  PROJ-131  Add Redis caching             est: 3h   [no deps]

  ⚠ 1.1 and 1.3 may both touch src/config/ → review before parallel

SERIES 2  (after series 1):
  2.1  PROJ-124  Frontend auth integration     est: 3h   [needs 1.1]
  2.2  PROJ-128  Update docs with new API      est: 1h   [needs 1.2]

📊 Series 1 wall-clock: 3h (vs 6h sequential = 2× speedup)
   Critical path: 1.1 → 2.1  (5h minimum)

────────────────────────────────────────────────
Analysis only. Worktree setup and task assignment are yours to manage.
Run with --apply-labels to write these labels onto the tracker (Step 5).

To start tracking once you're set up:
  /ferret:start PROJ-123 --estimate 2h
  /ferret:start PROJ-127 --estimate 1h    ← run in separate worktree/terminal
```

If a dependency **cycle** is detected, the planner reports the cycle members and labels
the rest of the graph; tell the user to break the cycle before re-running.

If `--json`: output an object keyed by numeric label, each value carrying
`{id, title, estimate, depends_on}`, plus top-level `critical_path` (ordered label list),
`speedup` (sequential_total / critical_path), and `conflicts`.

### Step 5 – Optional: apply labels to the tracker (`--apply-labels`)

Only runs when the user passes `--apply-labels`. This is the **only** path that writes
to an external system. Skip this step entirely otherwise.

1. **Build the rewrite.** For each labelled task, compute the new title:
   strip any existing numeric prefix, then prepend the label:
   ```
   new_title = label + " " + re.sub(r'^\d+(\.\d+){0,2}\s+', '', current_title)
   ```
   This is **idempotent**: a title already prefixed (`1.1 Implement auth service`)
   is rewritten, never stacked. If `new_title == current_title`, mark it a no-op.

2. **Show the diff and confirm.** Print every pending change as `before → after`
   (skip no-ops). Then ask for explicit confirmation. If the user declines, write nothing.

3. **`--dry-run`** short-circuits here: print the diff, then stop. No writes.

4. **Write** each non-no-op change via the tracker MCP:
   - **Jira:** `editJiraIssue` (update `summary`).
   - **Linear:** the Linear update-issue tool (update `title`).
   Report how many issues were updated and how many were skipped as no-ops.
