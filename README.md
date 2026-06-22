# Loop Engineering Development Workflow

This repository describes an AI-first development workflow for open-source
projects. It is designed for projects where human maintainers define intent and
review outcomes, while Codex and Paseo agents coordinate, implement, verify, and
deliver changes through standard GitHub primitives.

The workflow does not optimize for the traditional human developer experience.
It optimizes for agent reliability, resumability, parallel execution, and
quality gates.

## Core Idea

Loop engineering treats development as a set of feedback loops:

- Issues define work and persist project memory.
- Pull requests contain deliverable artifacts.
- PR reviews persist audit memory.
- CI verifies mechanical correctness.
- Audit agents verify intent, completeness, and code quality.
- Branch protection turns those checks into merge requirements.
- Paseo worktrees isolate concurrent implementation work.

The goal is to use existing infrastructure first. GitHub issues, pull requests,
comments, labels, checks, and protected branches are the system of record. Paseo
provides agent and worktree orchestration. Codex agents perform planning,
implementation, review, and coordination.

## Roles

The workflow uses three primary agent roles:

- Coordinator agent: owns task graph planning and scheduling.
- Implementer agent: owns one implementation branch and PR until CI is green.
- Auditor agent: reviews one CI-passing PR against its issue and project rules.

Human maintainers stay in the loop for intent, authorization, policy decisions,
and cases that agents cannot resolve safely.

## Minimal Workflow

For a simple issue:

```text
issue
  -> implementer worktree
  -> PR
  -> CI
  -> coordinator dispatches audit
  -> audit
  -> merge
```

For a complex issue:

```text
root issue
  -> coordinator creates a root branch
  -> coordinator splits child issues
  -> implementers work in parallel child branches
  -> child PRs merge into the root branch
  -> root PR merges to main after final CI and audit
```

## Documentation

- [Architecture](docs/architecture.md)
- [End-to-end workflow](docs/workflow.md)
- [Issue memory model](docs/issue-memory.md)
- [Quality gates](docs/quality-gates.md)
- [Paseo worktree model](docs/paseo-worktrees.md)
- [Event protocol](docs/event-protocol.md)
- [Repository structure](docs/repository-structure.md)
- [Coordinator state machine](docs/state-machines/coordinator.md)
- [Implementer state machine](docs/state-machines/implementer.md)
- [Auditor state machine](docs/state-machines/auditor.md)
- [Issue state machine](docs/state-machines/issue.md)
- [PR state machine](docs/state-machines/pr.md)
- [State labels reference](docs/reference/state-labels.md)

## Non-goals

The initial version intentionally avoids building a custom scheduler, database,
GitHub App, dashboard, or external memory service. Those can be added later only
when GitHub-native state and Paseo orchestration are no longer enough.
