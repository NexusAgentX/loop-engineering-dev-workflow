# Paseo Worktree Model

Paseo provides isolated workspaces for agents. This workflow relies on that
isolation to make parallel development safe.

## Worktree Rules

Each implementer should receive:

- one issue,
- one branch,
- one worktree,
- one PR.

The coordinator should record the agent id, branch, worktree or workspace id,
and PR in the issue.

## Branch Strategy

Simple issue:

```text
main
  <- PR from issue/<issue-number>-<slug>
```

Complex issue:

```text
main
  <- root PR from issue/<root-number>-<slug>
       <- child PR from issue/<child-number>-<slug>
       <- child PR from issue/<child-number>-<slug>
```

Child branches should be created from the root branch, not directly from `main`.

## Agent Lifetime

Implementer agents should remain resumable after opening a PR. When CI passes,
the implementer may stop active work, but the coordinator should still be able
to send a repair prompt to the same agent if audit fails.

The issue should record:

```md
Implementer agent: <agent-id>
Workspace id: <workspace-id>
Worktree: <path-if-known>
Branch: issue/123-example
PR: #456
State: waiting-audit
```

## Dispatching in Parallel

The coordinator can launch implementers in parallel only when tasks are
independent and isolated.

Parallel-safe examples:

- independent documentation files,
- separate modules,
- independent tests,
- non-overlapping templates,
- independent refactors with clear ownership boundaries.

Not parallel-safe examples:

- shared migration files,
- global lock files,
- one shared test database,
- one shared GUI session,
- one shared VM,
- one shared device,
- one shared port,
- changes to branch protection or repository settings.

## Merge Direction

Child PRs merge into the root branch. The root PR merges into `main`.

The coordinator must verify that child PRs target the correct base branch before
audit and merge. A child PR accidentally targeting `main` should be corrected or
closed and recreated.

## Cleanup

After merge, the coordinator should clean up branches and archived worktrees when
safe. Cleanup is never more important than preserving recoverability. If an
issue is still disputed or audit evidence is incomplete, keep the branch and
worktree references available.

