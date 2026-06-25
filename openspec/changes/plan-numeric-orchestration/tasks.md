## 1. Label algorithm (concurrency-planner agent)

- [x] 1.1 Add a "Numeric label assignment" section to `agents/concurrency-planner.md`: series = longest dependency depth + 1, track = position among independent tasks in a series, subtask = third level only for dedicated subtasks.
- [x] 1.2 Specify cycle detection: on a dependency cycle, report the members and refuse to label them instead of looping.
- [x] 1.3 Keep the existing critical-path / speedup algorithm and tie it to the new series layering.

## 2. Plan command output (commands/plan.md)

- [x] 2.1 Replace the "Phase N (parallel)" output section with the numeric scheme grouped by series, parallel tracks shown together.
- [x] 2.2 Update the `--json` output to key tasks by numeric label and include `critical_path` and `speedup`.
- [x] 2.3 Reaffirm the read-only-by-default contract in the command text (no files/sessions/writes on a plain run).

## 3. Opt-in label sync (commands/plan.md)

- [x] 3.1 Add `--apply-labels` and `--dry-run` argument parsing to the command instructions.
- [x] 3.2 Define the confirmation step: show the full before/after title diff and require explicit confirmation before any write.
- [x] 3.3 Define the idempotent prefix rewrite: strip leading `^\d+(\.\d+){0,2}\s+` then prepend the new label; skip the write when the label is unchanged.
- [x] 3.4 Wire the write to the tracker MCP tools (`editJiraIssue` for Jira, Linear update-issue), tracker-agnostic rewrite rule.

## 4. Docs

- [x] 4.1 Update the `/ferret:plan` row and commands overview in `CLAUDE.md` to describe the numeric scheme and the `--apply-labels` / `--dry-run` flags.

## 5. Verification

- [x] 5.1 Manual check: a graph with two independent tasks plus one dependent yields `1.1`, `1.2`, `2.1`; subtask level stays absent.
- [x] 5.2 Manual check: re-running `--apply-labels` on an already-prefixed title rewrites the prefix rather than stacking, and a stable label is a no-op.
