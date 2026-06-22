# End-to-End Workflow

This document describes how work moves from idea to merged PR.

## Workflow Levels

There are two levels:

- Simple workflow: one issue, one implementer, one PR.
- Coordinated workflow: one root issue, multiple child issues, root branch,
  multiple child PRs, final root PR.

The coordinator chooses the smallest workflow that can safely deliver the task.

## Simple Workflow

Use simple workflow when the issue is clear, isolated, and does not benefit from
parallel execution.

```text
issue
  -> implementer branch
  -> PR to main
  -> CI green
  -> audit
  -> merge
```

### Steps

1. A human or agent creates an issue with requirements and acceptance criteria.
2. The coordinator launches one implementer in a new Paseo worktree and branch.
3. The implementer completes the change and opens a PR to `main`.
4. The implementer waits for CI.
5. If CI fails, the implementer fixes the branch and waits again.
6. When CI passes, the implementer wakes the coordinator and records status.
7. The coordinator launches an auditor for the PR.
8. The auditor leaves a GitHub review with inline comments and a formal review
   outcome.
9. If the review requests changes, the coordinator marks the PR as not yet
   mergeable and the original implementer reads the PR review comments on
   GitHub to repair the branch.
10. If the review approves the PR, the coordinator merges the PR and closes the
    issue.

## Coordinated Workflow

Use coordinated workflow when the issue is large, multi-step, or parallelizable.

```text
root issue
  -> root branch
  -> child issues
  -> child implementers
  -> child PRs to root branch
  -> audits
  -> child merges
  -> root PR to main
  -> final audit
  -> merge
```

### Root Branch

The coordinator creates a root branch for the root issue:

```text
issue/<root-number>-<slug>
```

Child branches are created from the root branch:

```text
issue/<child-number>-<slug>
```

Child PRs target the root branch. The final root PR targets `main`.

### Task Graph

The coordinator records a task graph in the root issue:

```md
## Task Graph

- [ ] #101 Write workflow docs
  - State: running
  - Depends on: none
  - Branch: issue/101-workflow-docs
  - PR: pending

- [ ] #102 Add issue templates
  - State: ready
  - Depends on: none
  - Branch: pending
  - PR: pending

- [ ] #103 Final README pass
  - State: blocked
  - Depends on: #101, #102
  - Branch: pending
  - PR: pending
```

The task graph is the durable scheduler state. The coordinator may sleep or be
replaced at any time and recover from this graph.

## Wakeup Events

Agents should not busy-wait in the coordinator conversation. Each role wakes the
next responsible role only at decision points.

Important wakeup events:

- Implementer reports PR CI passed.
- Implementer reports blocked after exhausting local repair.
- Auditor submits a GitHub review with `APPROVE` or `REQUEST_CHANGES`.
- Human provides missing input.
- Merge conflict or branch protection failure requires reconciliation.

## Dispatch Rules

The coordinator may dispatch multiple implementers in parallel when:

- each target issue is `ready`,
- all dependencies are merged or otherwise satisfied,
- each implementer has an isolated branch and worktree,
- the tasks do not compete for a shared exclusive resource.

The coordinator must serialize tasks when they touch shared runtime resources,
global configuration, shared ports, single-user UI state, external mutable test
environments, or other exclusive resources.

## Return-to-Implementer Rule

If audit fails, the coordinator should return the task to the original
implementer whenever possible. The original implementer already owns the branch,
PR, and local context. The coordinator does not need to relay full review
findings because the implementer can read GitHub review comments directly.

Only create a replacement repair agent when:

- the original implementer no longer exists,
- its worktree is unavailable,
- the agent is blocked,
- or the coordinator determines that a fresh context is safer.

## Done Criteria

A simple issue is done when:

- its PR is merged to `main`,
- the linked issue is closed,
- CI and audit evidence are present.

A complex root issue is done when:

- all required child issues are closed,
- all child PRs are merged into the root branch or intentionally cancelled,
- the root PR is merged to `main`,
- final CI and final audit evidence are present.
