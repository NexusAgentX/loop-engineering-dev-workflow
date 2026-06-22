# Issue State Machine

Issues are task records and memory records. Their state should be readable by
humans and agents.

## State Diagram

```text
new
  -> triage
  -> ready
  -> running
  -> pr-open
  -> ready-for-audit
  -> audit-running
  -> audit-failed
  -> running
  -> ready-for-audit
  -> audit-passed
  -> merged
  -> closed
```

Blocked path:

```text
triage -> needs-info
ready -> blocked
running -> blocked
blocked -> ready
```

Cancelled path:

```text
new | triage | ready | running -> cancelled
```

## States

### new

The issue exists but has not been triaged.

### triage

An agent or maintainer clarifies scope, acceptance criteria, and priority.

### needs-info

The issue lacks required information.

### ready

The issue has enough information to implement and all dependencies are
satisfied.

### blocked

The issue cannot proceed because of dependencies, missing input, resource
conflict, or external failure.

### running

An implementer owns the issue.

### pr-open

The implementer opened a PR, but CI may still be pending or failing.

### ready-for-audit

The PR has passed required CI and is waiting for coordinator-dispatched audit.

### audit-running

An auditor is reviewing the PR.

### audit-failed

Audit found required fixes. The coordinator should return the issue to the
original implementer.

### audit-passed

The auditor passed the PR. The coordinator may merge if branch protection allows
it.

### merged

The PR was merged into its target branch.

### closed

The issue is complete and closed.

### cancelled

The issue is intentionally not being delivered.

## Root Issue Additions

A root issue may have child issues. It is not complete until required child work
is complete and the root PR merges.

Root issue states:

```text
planning
  -> dispatching
  -> waiting-children
  -> integrating
  -> final-audit
  -> merged
  -> closed
```

## Issue Invariants

- Every running issue has an owner.
- Every PR-backed issue links to its PR.
- Every blocked issue names the blocker.
- Every audit failure lists required fixes.
- Every closed issue links to merged evidence or cancellation rationale.

