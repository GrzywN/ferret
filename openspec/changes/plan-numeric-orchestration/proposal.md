## Why

`/ferret:plan` today emits a prose "Phase 1 / Phase 2" report that is slow to produce and awkward to act on — the user still has to mentally map phases to "what can I run right now". A flat, numeric labelling scheme (`1`, `1.1`, `1.2`, `2.1`) communicates orchestration at a glance: the first number is the sequential series, the second is a parallel track inside that series. The command should stay a zero-side-effect advisor by default, but optionally write those labels back onto Jira/Linear issues so the plan lives where the work lives.

## What Changes

- **Numeric orchestration labels** replace the "Phase N (parallel)" grouping. Scheme:
  - `N` — sequential series. `1` must finish before `2` starts.
  - `N.M` — parallel track inside series `N`. All `1.*` (`1.1`, `1.2`) can run simultaneously.
  - `N.M.K` — subtask split inside a track. Used **only** when an issue has dedicated subtasks that need separate assignment; normally absent.
- **Lower friction by default**: the plan is a fast, read-only report. No worktrees, no sessions, no files written. (Reaffirms existing contract, removes the heavier prose ceremony.)
- **Opt-in label sync** (`--apply-labels`): when explicitly requested, prefix Jira/Linear issue titles with their numeric label (e.g. `Implement auth service` → `1.1 Implement auth service`). Requires confirmation, is idempotent (re-running rewrites existing prefixes, never stacks them), and supports `--dry-run`.
- `/ferret:plan` output and the `concurrency-planner` agent are updated to compute and render the numeric scheme instead of named phases.

## Capabilities

### New Capabilities
- `numeric-orchestration`: read-only computation and rendering of the `series.parallel.subtask` numeric labels from a task dependency graph, including critical-path and speedup summary.
- `task-label-sync`: optional, opt-in writing of numeric labels as Jira/Linear issue-title prefixes, with confirmation, dry-run, and idempotent re-labelling.

### Modified Capabilities
<!-- None — no existing openspec specs. The legacy commands/plan.md is reworked, captured under the new specs above. -->

## Impact

- `commands/plan.md` — output section reworked to numeric scheme; new `--apply-labels` / `--dry-run` args.
- `agents/concurrency-planner.md` — label-assignment algorithm added (map dependency layers → series, independent tasks in a layer → parallel tracks, subtasks → third level).
- `CLAUDE.md` — update the `/ferret:plan` row and commands overview to describe the numeric scheme.
- Jira/Linear MCP write tools (`editJiraIssue`, Linear update-issue) used only on the opt-in path.
