# Documentation Index

This directory defines the Loop Engineering Development Workflow. The documents
are written as a protocol for AI agents and maintainers, not as a generic
developer handbook.

## Start Here

- [Architecture](architecture.md) explains the system model.
- [End-to-end workflow](workflow.md) explains simple and complex task flow.
- [Issue memory model](issue-memory.md) explains why issues are the durable
  context store.
- [Quality gates](quality-gates.md) explains CI and agent audit responsibilities.
- [Paseo worktree model](paseo-worktrees.md) explains branch and worktree
  isolation.
- [Event protocol](event-protocol.md) explains how agents wake each other.
- [Repository structure](repository-structure.md) describes the intended
  template layout.

## State Machines

- [Coordinator](state-machines/coordinator.md)
- [Implementer](state-machines/implementer.md)
- [Auditor](state-machines/auditor.md)
- [Issue](state-machines/issue.md)
- [Pull request](state-machines/pr.md)

## Reference

- [State labels](reference/state-labels.md)
