# Auditor State Machine

The auditor agent reviews a CI-passing PR. It checks whether the PR satisfies the
issue and project rules.

The auditor does not modify implementation code. It writes the review directly
to the GitHub PR, including inline comments when needed.

## State Diagram

```text
assigned
  -> collect-context
  -> review-diff
  -> review-validation
  -> submit-github-review
  -> notify-coordinator
```

Blocked path:

```text
collect-context -> blocked -> notify-coordinator
```

## States

### assigned

The coordinator assigns a PR, linked issue, and audit scope.

### collect-context

The auditor reads:

- issue requirements,
- acceptance criteria,
- PR description,
- diff,
- CI results,
- relevant project rules,
- prior PR reviews if any.

### review-diff

The auditor inspects changed files for correctness, scope, maintainability, and
hidden shortcuts.

### review-validation

The auditor verifies that tests and validation evidence match the risk of the
change.

### submit-github-review

The auditor submits the GitHub review:

- `APPROVE` when the PR can merge,
- `REQUEST_CHANGES` when fixes are required,
- `COMMENT` when it has notes but no blocking decision.

It also leaves inline comments on exact lines or files when useful.

### blocked

The auditor cannot complete review because required context is missing or GitHub
state is inconsistent.

### notify-coordinator

The auditor wakes the coordinator with the PR review result and a link to the
review.

## Auditor Invariants

- Do not edit the PR branch.
- Do not approve work without checking the linked issue.
- Do not treat CI success as sufficient.
- Prefer concrete file and behavior findings.
- Separate blocking issues from optional improvements.
- Record residual risk even on pass.
- Put findings in the GitHub review so implementers can read them directly.
