# Event Protocol

The workflow is event-driven. Agents should not keep a coordinator busy while
waiting for CI, audit, or human input. Instead, each role wakes the next
responsible role when a decision point is reached.

Events are durable only after they are written to GitHub state. A chat message or
Paseo notification is a wakeup mechanism, not the source of truth.

## Event Types

### coordinator.assigned-implementation

Sent by coordinator to an implementer.

```md
Event: coordinator.assigned-implementation
Issue: #123
Branch: issue/123-example
Base: main | issue/100-root
Expected PR base: main | issue/100-root
Acceptance criteria:
- ...
```

The implementer should create or use the assigned worktree and branch, then own
the task through CI success.

### implementer.pr-opened

Written by implementer when a PR is opened.

```md
Event: implementer.pr-opened
Issue: #123
PR: #456
Branch: issue/123-example
Base: issue/100-root
State: ci-wait
```

This does not wake the coordinator unless the repository chooses to track PR
creation explicitly. The implementer still owns CI.

### implementer.ci-passed

Sent by implementer to coordinator after required CI checks pass.

```md
Event: implementer.ci-passed
Issue: #123
PR: #456
Branch: issue/123-example
CI: passed
State: ready-for-audit
Validation:
- ...
Risks:
- ...
```

This is the normal trigger for coordinator audit dispatch.

### implementer.blocked

Sent by implementer to coordinator when it cannot safely continue.

```md
Event: implementer.blocked
Issue: #123
PR: #456 | none
Reason: ...
Attempts:
- ...
Needed decision:
- ...
```

The coordinator decides whether to clarify, reassign, split, serialize, or ask a
human.

### coordinator.assigned-audit

Sent by coordinator to auditor after CI passes.

```md
Event: coordinator.assigned-audit
Issue: #123
PR: #456
Audit scope:
- Check issue acceptance criteria.
- Check changed files.
- Check CI evidence.
- Check project rules.
Expected GitHub review: APPROVE | REQUEST_CHANGES | COMMENT
```

The auditor should not modify the branch. It should submit a GitHub PR review
instead.

### auditor.audit-passed

Sent by auditor to coordinator when a GitHub review with `APPROVE` is
submitted.

```md
Event: auditor.audit-passed
Issue: #123
PR: #456
Result: pass
GitHub review: APPROVE
Evidence:
- ...
Residual risk:
- ...
```

The coordinator may merge if branch protection, base branch checks, and review
state all pass.

### auditor.audit-failed

Sent by auditor to coordinator when a GitHub review with `REQUEST_CHANGES` is
submitted.

```md
Event: auditor.audit-failed
Issue: #123
PR: #456
Result: fail
GitHub review: REQUEST_CHANGES
Review: <GitHub review URL>
Evidence:
- ...
```

The coordinator should return the task to the original implementer, but the
implementer should read the review comments directly from GitHub instead of the
coordinator retransmitting them.

### coordinator.returned-for-repair

Sent by coordinator to the implementer after audit failure.

```md
Event: coordinator.returned-for-repair
Issue: #123
PR: #456
Audit result: fail
Review: <GitHub review URL>
Next state: implementer owns CI again
```

The implementer reads the GitHub review comments, repairs the PR, and repeats
the CI loop.

### coordinator.merged

Written by coordinator after a successful merge.

```md
Event: coordinator.merged
Issue: #123
PR: #456
Merged into: main | issue/100-root
Next unblocked issues:
- #124
- #125
```

After this event, the coordinator scans the task graph and dispatches newly ready
work.

## Durable Event Storage

Each event should be written to at least one durable place:

- linked issue comment,
- PR comment,
- issue body task graph,
- PR body status section,
- label or check status when available.

The issue body should hold the current state. Comments can hold the event log.

## Wakeup Discipline

The coordinator should wake only when it needs to decide something:

- CI passed and audit should be launched,
- audit finished and merge or repair should be chosen,
- a blocker needs scheduling or human input,
- a merge changed dependency state.

The implementer should not wake the coordinator for ordinary CI failures that it
can repair itself.

## Event Idempotency

Agents may be restarted or duplicated. Events should be safe to process more
than once.

Before acting on an event, the coordinator should refresh GitHub state and ask:

- Is this issue still in the expected state?
- Is this PR still open?
- Did CI still pass?
- Has audit already been dispatched?
- Has this PR already merged?

If the event is stale, record that it was ignored and continue from current
GitHub state.
