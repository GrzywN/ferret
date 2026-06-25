## Context

`/ferret:plan` is a prompt-defined command (`commands/plan.md`) backed by the `concurrency-planner` agent. It fetches tasks from Jira/Linear MCP (or a manual list), builds a dependency graph, and reports parallelization. Today it renders named "Phase N (parallel)" groups in prose. The user finds this slow and hard to act on, and wants two things: a terse numeric labelling scheme, and an optional path to push those labels into the tracker. There is no code to change beyond the markdown command/agent definitions — the "implementation" is the instructions the model follows.

## Goals / Non-Goals

**Goals:**
- Replace phase naming with a `series.parallel.subtask` numeric scheme that maps directly to "what can I run now".
- Keep the default run zero-side-effect and fast.
- Add an explicit, confirmed, idempotent opt-in to write labels onto Jira/Linear issue titles.

**Non-Goals:**
- Creating worktrees, branches, or sessions (still the user's responsibility / other commands).
- Inventing dependency data the tracker doesn't have — inference stays best-effort, as today.
- A persistent label store or reconciliation; labels live only in the report and (optionally) issue titles.

## Decisions

**Label assignment = dependency layers.** Compute the label from the graph:
1. Topologically sort; assign each task a *series* = its longest-dependency-depth + 1 (so all of a task's blockers sit in lower series). This guarantees the spec's "B's series > A's series when B depends on A".
2. Within a series, each independent task gets the next *track* number `.M`. Tasks in the same series are by construction dependency-free relative to each other, so they are parallel-safe.
3. *Subtask* `.K` is emitted only when an issue carries dedicated child subtasks needing separate assignment — otherwise omit the third level.

Alternative considered: keep named phases and just add numbers — rejected, the user explicitly wants the numeric scheme to *be* the interface.

**Series = longest path, not shortest.** Using longest dependency depth (not first-available slot) keeps a task in the same series as its true earliest-possible start while never placing it before a blocker. This is the standard critical-path layering and makes the speedup number honest.

**Opt-in write is a separate, gated step.** `--apply-labels` is the only trigger; it always shows the diff and asks for confirmation; `--dry-run` short-circuits before any write. This keeps the default contract (read-only) intact and contains the only risky operation behind two gates.

**Idempotency via prefix regex.** Before writing, strip any leading `^\d+(\.\d+){0,2}\s+` from the current title, then prepend the new label. Re-running rewrites rather than stacks, and a stable label yields an unchanged title (skip the write). This is the one piece of non-trivial logic and gets a check.

## Risks / Trade-offs

- **Bad/missing dependency data → wrong series** → Mitigation: inference stays best-effort and the report shows the inferred dependencies so the user can sanity-check before `--apply-labels`.
- **Title-prefix writes are user-visible in the tracker and mildly destructive** → Mitigation: opt-in flag + confirmation + dry-run + idempotent rewrite (no stacking).
- **Cycles in the dependency graph break topological sort** → Mitigation: detect cycles, report them, and refuse to label the cycle members rather than loop.
- **Two trackers (Jira/Linear) with different title-edit APIs** → Mitigation: the rewrite rule is tracker-agnostic; only the MCP write call differs (`editJiraIssue` vs Linear update).

## Migration Plan

No data migration. Update `commands/plan.md`, `agents/concurrency-planner.md`, and the `CLAUDE.md` overview together. Existing behaviour (fetch, graph, manual fallback, `--json`) is preserved; only the output rendering and the new opt-in flags are added. Rollback = revert the three markdown files.

## Open Questions

- For Linear, prefixing the *title* vs using a dedicated label/field — title prefix chosen for parity with Jira and visibility, but a Linear label could be cleaner. Revisit if users object.
