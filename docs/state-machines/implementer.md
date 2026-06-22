# Implementer State Machine

The implementer agent owns one task branch and one PR until CI passes.

The implementer should not delegate audit and should not mark its own work as
accepted. Its responsibility ends at `ready-for-audit`, unless the coordinator
returns the task for repair.

## State Diagram

```text
assigned
  -> prepare-worktree
  -> implement
  -> open-pr
  -> wait-ci
  -> ci-failed
  -> implement
  -> wait-ci
  -> ci-passed
  -> notify-coordinator
  -> waiting-audit
```

Repair path:

```text
waiting-audit
  -> audit-failed
  -> implement
  -> wait-ci
  -> ci-passed
  -> notify-coordinator
```

Blocked path:

```text
implement -> blocked -> notify-coordinator
```

## States

### assigned

The coordinator assigns an issue, branch, base branch, and expected PR target.

### prepare-worktree

The implementer verifies it is in the correct worktree and branch. It checks
repository state before editing.

### implement

The implementer makes the required change. It should keep the scope aligned with
the issue and avoid unrelated refactors.

### open-pr

The implementer opens or updates a PR. The PR must link the issue and include
verification notes.

### wait-ci

The implementer waits for required CI checks.

### ci-failed

The implementer reads CI logs, fixes the branch, pushes again, and returns to
`wait-ci`.

The coordinator is not woken for normal CI failures.

### ci-passed

Required CI checks passed. The implementer writes a concise status update to the
issue or PR.

### notify-coordinator

The implementer wakes the coordinator and includes:

- issue number,
- PR number,
- branch,
- CI status,
- validation summary,
- any residual risks.

### waiting-audit

The implementer stops active work but remains resumable. The coordinator may send
repair instructions if audit fails.

### audit-failed

The coordinator returns audit findings. The implementer fixes them and owns CI
again.

### blocked

The implementer cannot continue safely. It records the blocker and wakes the
coordinator.

## Implementer Invariants

- Own CI until green.
- Do not wake the coordinator for every CI retry.
- Do not self-approve.
- Do not merge.
- Do not edit outside scope without recording why.
- Keep issue and PR state current enough for recovery.

