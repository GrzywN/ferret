## ADDED Requirements

### Requirement: Opt-in label writing

The system SHALL write numeric labels back to Jira/Linear issue titles only when the user explicitly passes `--apply-labels`. Without that flag, no external issue SHALL be modified.

#### Scenario: Default run does not write

- **WHEN** `/ferret:plan` runs without `--apply-labels`
- **THEN** no Jira/Linear issue title is changed

#### Scenario: Explicit opt-in writes labels

- **WHEN** the user runs `/ferret:plan --apply-labels` and confirms
- **THEN** each issue's title is prefixed with its numeric label (e.g. `Implement auth service` becomes `1.1 Implement auth service`)

### Requirement: Confirmation before writing

The system SHALL present the full set of intended title changes and SHALL obtain explicit user confirmation before performing any write.

#### Scenario: User declines

- **WHEN** the user is shown the pending title changes and declines
- **THEN** no issue is modified

### Requirement: Dry-run preview

The system SHALL support `--dry-run`, which prints the exact before/after title for each issue without performing any write.

#### Scenario: Dry-run performs no writes

- **WHEN** `/ferret:plan --apply-labels --dry-run` is run
- **THEN** the before/after titles are printed and no issue is modified

### Requirement: Idempotent re-labelling

When an issue title already carries a numeric-label prefix, the system SHALL replace that existing prefix rather than prepend a second one, so re-running never stacks labels.

#### Scenario: Re-running updates instead of stacking

- **WHEN** an issue titled `1.1 Implement auth service` is re-labelled to track `2.1`
- **THEN** the resulting title is `2.1 Implement auth service`, not `2.1 1.1 Implement auth service`

#### Scenario: Stable label is a no-op write

- **WHEN** re-running produces the same label an issue already has
- **THEN** that issue's title is left unchanged
