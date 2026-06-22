# Architecture

Loop Engineering is an AI-first repository workflow built on GitHub, Git, Codex,
and Paseo. It assumes that agents, not humans, perform most operational
development work.

The architecture is intentionally conservative:

- GitHub is the durable state store.
- Git branches and worktrees isolate execution.
- Pull requests are the only mergeable artifact.
- CI provides automated quality signals.
- Audit agents provide intent and quality review.
- Branch protection enforces the rules.
- Paseo manages agent lifecycles and worktree placement.

## Design Principles

### Agent-first

Repository structure, templates, and process documents should be optimized for
agent execution. A human may read them, but the primary reader is an agent that
needs clear state, precise responsibilities, and recoverable instructions.

### GitHub-native

Use GitHub issues, pull requests, labels, comments, checks, and branch
protection before adding new infrastructure. GitHub is already the collaboration
surface for open-source projects and already provides durable objects.

### Issue as memory

Agent context windows are limited and long-running work may cross context
compaction boundaries. Issues must therefore hold the durable task memory:
requirements, decisions, task graph, status, PRs, validation evidence, and
handoff notes.

### PR as artifact

A pull request is the reviewable and mergeable output of agent work. Agent work
that does not end in a PR is not considered delivered.

### Concurrency with isolation

Parallel work should use isolated Paseo worktrees and branches. Agents should
not share mutable workspaces unless that sharing is explicit and serialized.

### Quality before merge

No PR merges only because an implementer says it is done. A merge requires CI
success and agent audit success. Branch protection should encode this as a hard
rule.

### No unnecessary entities

Simple issues should use a simple path. Complex coordinator/root-branch flows are
reserved for work that benefits from splitting or parallel execution.

## System Components

### Human maintainer

The maintainer defines intent, sets repository policy, authorizes risky actions,
and intervenes when agents are blocked or when judgment cannot be safely
delegated.

### Coordinator agent

The coordinator owns planning and scheduling for a root issue. It builds or
updates the task graph, dispatches implementer agents, dispatches auditor agents,
merges passing PRs, and records durable state.

The coordinator does not write implementation code unless the issue is too small
to justify delegation.

### Implementer agent

An implementer owns one implementation branch and one PR. It edits code or docs,
opens the PR, watches CI, fixes CI failures, and only wakes the coordinator when
the PR has passed CI.

### Auditor agent

An auditor reviews a CI-passing PR against its issue, the repository rules, and
the actual diff. It does not modify code. It reports pass or fail with evidence.

### GitHub issue

An issue is the durable memory object for a task. For complex work, the root
issue contains the task graph and links to child issues.

### GitHub PR

A PR is the deliverable artifact. Child PRs may target a root branch. Root PRs
target `main`.

### CI

CI runs deterministic checks. It should be project-specific but must report
clear pass/fail status to GitHub checks.

### Branch protection

Branch protection prevents direct pushes to `main` and requires the configured
checks before merge.

## Simple Task Flow

Use this when a task does not need splitting:

```text
issue
  -> coordinator or human launches implementer
  -> implementer creates branch and worktree
  -> implementer opens PR
  -> implementer fixes until CI passes
  -> implementer wakes coordinator
  -> coordinator launches auditor
  -> auditor passes or fails
  -> coordinator merges or returns work
```

## Complex Task Flow

Use this when a task benefits from planning, dependency management, or parallel
execution:

```text
root issue
  -> coordinator creates root branch
  -> coordinator creates or updates child issues
  -> coordinator records dependency graph
  -> coordinator launches ready child implementers in parallel
  -> child implementers open PRs against root branch
  -> coordinator dispatches audits after child CI passes
  -> coordinator merges passing child PRs into root branch
  -> coordinator launches newly unblocked tasks
  -> coordinator opens or updates root PR to main
  -> final CI and audit pass
  -> coordinator merges root PR
```

## Source of Truth

When state conflicts, prefer sources in this order:

1. GitHub PR merge status and checks.
2. GitHub issue task graph and latest coordinator comment.
3. Git branch history and worktree state.
4. Agent memory or chat context.

Agent memory is useful but not authoritative.

## Recovery Model

Any agent should be replaceable. A replacement agent must be able to recover by
reading:

- the root issue,
- linked child issues,
- linked PRs,
- latest CI results,
- latest audit comments,
- branch names,
- worktree or workspace identifiers when available.

If recovery is not possible from GitHub state, the workflow has failed its
primary design goal.

