# Pull Request State Machine

A PR is the mergeable artifact of agent work.

## State Diagram

```text
draft
  -> open
  -> ci-wait
  -> ci-failed
  -> ci-wait
  -> ci-passed
  -> audit-wait
  -> audit-running
  -> audit-failed
  -> ci-wait
  -> ci-passed
  -> audit-running
  -> audit-passed
  -> mergeable
  -> merged
```

Exception paths:

```text
mergeable -> merge-conflict
mergeable -> branch-protection-blocked
open -> closed-without-merge
```

## States

### draft

The PR exists but is not ready for CI or review.

### open

The PR is open and expected to move through checks.

### ci-wait

Required CI checks are pending.

### ci-failed

One or more required checks failed. The implementer owns repair.

### ci-passed

Required checks passed. The implementer wakes the coordinator.

### audit-wait

The coordinator has not yet dispatched audit.

### audit-running

An auditor is reviewing the PR and preparing a GitHub review.

### audit-failed

Audit failed. The PR has `REQUEST_CHANGES`, and the implementer should read the
review comments on GitHub.

### audit-passed

Audit passed. The PR has `APPROVE` and can move toward merge.

### mergeable

CI and audit passed, base branch is correct, and GitHub reports the PR can merge.

### merge-conflict

The branch cannot merge cleanly. The coordinator assigns repair.

### branch-protection-blocked

Branch protection prevents merge. The coordinator reconciles missing checks,
reviews, labels, or settings.

### merged

The PR was merged.

### closed-without-merge

The PR was intentionally abandoned.

## Base Branch Rules

Simple PRs target `main`.

Child PRs in coordinated work target the root branch.

Root PRs target `main`.

The coordinator must verify the base branch before audit and merge.

## PR Invariants

- Every PR links to an issue.
- Every PR explains validation.
- Every PR has a clear base branch.
- Implementers own CI failure repair.
- Auditors own audit result.
- GitHub review state is the durable audit result.
- Coordinators own merge decisions.
