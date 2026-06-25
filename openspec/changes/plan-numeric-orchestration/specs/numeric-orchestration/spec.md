## ADDED Requirements

### Requirement: Numeric label scheme

The system SHALL assign each task a hierarchical numeric label of the form `N`, `N.M`, or `N.M.K` where:
- `N` (series) is the sequential execution order. All tasks labelled `N` MUST be complete before any task labelled `N+1` begins.
- `M` (track) distinguishes tasks that can run in parallel within the same series `N`. Tasks `N.1` and `N.2` SHALL have no dependency between them and MAY run simultaneously.
- `K` (subtask) splits a single track into ordered or parallel subtasks. The third level SHALL be emitted only when an issue has dedicated subtasks that need separate assignment; otherwise labels stop at two levels.

#### Scenario: Independent tasks become parallel tracks

- **WHEN** the dependency graph has two tasks in the same series with no dependency between them
- **THEN** they receive labels sharing the same series number and different track numbers (e.g. `1.1` and `1.2`)

#### Scenario: Dependent task advances the series

- **WHEN** task B depends on task A
- **THEN** B's series number is strictly greater than A's series number

#### Scenario: Subtask level omitted by default

- **WHEN** no task carries dedicated subtasks
- **THEN** every emitted label has at most two numeric components

#### Scenario: Subtasks produce a third level

- **WHEN** an issue has dedicated subtasks requiring separate assignment
- **THEN** those subtasks receive a third-level label under their parent's `N.M` (e.g. `1.1.1`, `1.1.2`)

### Requirement: Read-only by default

The command SHALL NOT create files, worktrees, sessions, or mutate any external system when producing the orchestration plan. Its default output is advisory only.

#### Scenario: No side effects on a plain run

- **WHEN** `/ferret:plan` runs without an opt-in write flag
- **THEN** no file is written, no session is created, and no Jira/Linear issue is modified

### Requirement: Orchestration report

The system SHALL render a report grouping tasks by series, listing parallel tracks within each series, and SHALL include each task's estimate, dependencies, the critical path, and the sequential-vs-parallel speedup.

#### Scenario: Report groups by series and track

- **WHEN** the plan is rendered for a set of labelled tasks
- **THEN** tasks appear grouped by series number, parallel tracks are shown together, and the critical path and speedup are summarised

#### Scenario: JSON output requested

- **WHEN** `/ferret:plan --json` is run
- **THEN** the system outputs a JSON object keyed by numeric label including each task's dependencies, plus `critical_path` and `speedup` fields
