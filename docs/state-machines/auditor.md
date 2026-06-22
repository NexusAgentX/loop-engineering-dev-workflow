# Auditor State Machine

The auditor agent reviews a CI-passing PR. It checks whether the PR satisfies the
issue and project rules.

The auditor does not modify implementation code.

## State Diagram

```text
assigned
  -> collect-context
  -> review-diff
  -> review-validation
  -> decide
  -> report-pass
  -> notify-coordinator
```

Failure path:

```text
decide
  -> report-fail
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
- prior audit comments if any.

### review-diff

The auditor inspects changed files for correctness, scope, maintainability, and
hidden shortcuts.

### review-validation

The auditor verifies that tests and validation evidence match the risk of the
change.

### decide

The auditor decides `pass` or `fail`.

Pass means the PR can be merged if branch protection allows it. Fail means the
coordinator must return the task to implementation.

### report-pass

The auditor writes evidence for why the PR can merge.

### report-fail

The auditor writes concrete findings and required fixes. Findings should be
specific enough for the implementer to act without guessing.

### blocked

The auditor cannot complete review because required context is missing or GitHub
state is inconsistent.

### notify-coordinator

The auditor wakes the coordinator with the result and a link to its audit
comment.

## Auditor Invariants

- Do not edit the PR branch.
- Do not approve work without checking the linked issue.
- Do not treat CI success as sufficient.
- Prefer concrete file and behavior findings.
- Separate blocking issues from optional improvements.
- Record residual risk even on pass.

