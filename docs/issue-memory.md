# Issue Memory Model

Issues are the durable memory layer of the workflow.

Agent context is temporary. Issue state is durable, inspectable, linkable, and
recoverable. Every agent role must treat issues as the task board and notebook.

## What Belongs in an Issue

Each issue should contain:

- problem statement,
- desired outcome,
- acceptance criteria,
- relevant context,
- known constraints,
- dependencies,
- implementation plan or current plan,
- task checklist,
- linked branches and PRs,
- agent ownership,
- validation evidence,
- audit results,
- blocking questions,
- final outcome.

## What Does Not Belong Only in Agent Context

These facts must not live only in an agent conversation:

- why a task was split,
- why a dependency exists,
- which PR implements the issue,
- what tests passed,
- what audit found,
- what remains blocked,
- which assumptions were made,
- how to resume after interruption.

If a future agent needs it to continue work, write it to the issue.

## Root Issue Fields

For coordinated work, the root issue should include:

```md
## Objective

## Acceptance Criteria

## Constraints

## Task Graph

## Coordinator State

## Branches and PRs

## Decisions

## Validation

## Open Questions
```

## Child Issue Fields

Child issues should include:

```md
## Parent

## Scope

## Acceptance Criteria

## Dependencies

## Assigned Implementer

## Branch and PR

## CI Status

## Audit Status

## Notes
```

## Comment Protocol

Agents should add comments for state transitions that matter:

- coordinator planned or updated task graph,
- implementer opened PR,
- implementer reached CI passed,
- implementer became blocked,
- auditor passed or failed,
- coordinator merged PR,
- coordinator returned work for repair.

Avoid noisy comments for every minor edit. The issue should remain readable as a
state log.

## Recovery Contract

A replacement coordinator should be able to recover by reading the root issue and
linked PRs. A replacement implementer should be able to recover by reading its
issue, branch, PR, and latest feedback. A replacement auditor should be able to
recover by reading the issue, PR, diff, CI logs, and repository rules.

