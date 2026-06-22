# Coordinator State Machine

The coordinator agent owns planning, scheduling, reconciliation, audit dispatch,
merge decisions, and task graph updates.

The coordinator should not busy-wait on CI. Implementers own CI until green.

## State Diagram

```text
idle
  -> intake
  -> plan
  -> dispatch-ready
  -> sleeping
  -> reconcile
  -> dispatch-audit
  -> waiting-audit
  -> reconcile
  -> merge-ready
  -> scan-graph
  -> dispatch-ready | sleeping | done
```

Failure and exception paths:

```text
plan -> blocked
dispatch-ready -> blocked
reconcile -> return-to-implementer
reconcile -> human-escalation
merge-ready -> merge-conflict
merge-ready -> branch-protection-blocked
```

## States

### idle

The coordinator has not accepted a root issue or has finished all assigned work.

### intake

The coordinator reads the root issue, linked issues, existing branches, existing
PRs, and project rules.

### plan

The coordinator decides whether the issue is simple or coordinated.

For coordinated work, it creates or updates:

- root branch,
- child issues,
- dependency graph,
- task states,
- acceptance criteria per child issue.

### dispatch-ready

The coordinator launches implementer agents for every ready, unblocked task that
can safely run in parallel.

For each implementer, it records:

- issue,
- agent id,
- branch,
- workspace id or worktree,
- expected PR base.

### sleeping

The coordinator has no immediate decision to make. It waits to be woken by an
implementer, auditor, human, or repository event.

### reconcile

The coordinator refreshes GitHub state and compares it to issue memory.

It checks:

- child issue states,
- PR base branches,
- CI status,
- audit status,
- mergeability,
- dependency graph,
- blocker comments.

### dispatch-audit

If a PR has passed CI and has not yet passed audit, the coordinator launches an
auditor agent.

The auditor receives:

- linked issue,
- PR URL or number,
- acceptance criteria,
- project rules,
- expected output format.

### waiting-audit

The coordinator waits for the auditor to finish. The auditor must wake the
coordinator with pass or fail.

### return-to-implementer

If audit fails, the coordinator sends the audit findings back to the original
implementer. The implementer resumes the branch, fixes the PR, and again owns CI
until it passes.

### merge-ready

If CI and audit pass, the coordinator merges the PR into its target branch.

For child PRs, the target is the root branch. For the root PR or simple PR, the
target is `main`.

### scan-graph

After any merge, the coordinator scans the task graph for newly unblocked tasks.

### done

The root issue is complete. The final PR is merged, required child issues are
closed, and evidence is recorded.

## Wakeup Inputs

The coordinator can be woken by:

- implementer: PR CI passed,
- implementer: blocked,
- auditor: audit passed,
- auditor: audit failed,
- human: new requirements or approval,
- GitHub: merge conflict or branch protection state changed.

## Coordinator Invariants

- Never launch audit before required CI passes.
- Never merge before CI and audit both pass.
- Never rely on chat-only state for scheduling.
- Never dispatch two agents to the same branch unless intentionally repairing.
- Prefer returning failed audit work to the original implementer.
- Record every durable state transition in the issue or PR.

