# Repository Structure

The repository should provide enough structure for agents to operate
consistently without assuming a programming language.

This document describes the intended template layout. The initial documentation
phase does not require every file to exist yet.

## Target Layout

```text
.
├── README.md
├── AGENTS.md
├── docs/
│   ├── README.md
│   ├── architecture.md
│   ├── workflow.md
│   ├── issue-memory.md
│   ├── quality-gates.md
│   ├── paseo-worktrees.md
│   ├── event-protocol.md
│   ├── repository-structure.md
│   ├── state-machines/
│   │   ├── coordinator.md
│   │   ├── implementer.md
│   │   ├── auditor.md
│   │   ├── issue.md
│   │   └── pr.md
│   └── reference/
│       └── state-labels.md
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug.yml
│   │   ├── idea.yml
│   │   └── task.yml
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── workflows/
│       └── ci.yml
└── .agents/
    └── skills/
        ├── coordinator/
        │   └── SKILL.md
        ├── implementer/
        │   └── SKILL.md
        └── auditor/
            └── SKILL.md
```

## Root Files

### README.md

Public entry point. It should explain what the workflow is, what problem it
solves, and where to find the detailed protocol.

### AGENTS.md

Agent-facing repository contract. It should be short, direct, and stable. It
should not duplicate all workflow documentation. It should point agents to the
right detailed docs.

## docs/

The docs directory is the source of truth for process and architecture.

Detailed docs should be split by topic. Avoid one giant document that mixes
architecture, role contracts, state machines, templates, and repository setup.

## .github/

The `.github` directory should encode GitHub-native collaboration:

- issue templates create structured issue memory,
- PR template requires evidence and links,
- workflows run CI and repository checks,
- branch protection should be configured in GitHub settings.

The template should remain language-neutral. A downstream project can add
language-specific checks.

## .agents/

The `.agents` directory should contain agent role skills or prompt contracts.
These should make the documented workflow executable by Codex and Paseo agents.

Suggested split:

- coordinator skill,
- implementer skill,
- auditor skill.

Each skill should define role responsibilities, inputs, outputs, state updates,
and forbidden actions.

## What This Template Should Not Contain by Default

Do not add language-specific package managers, build tools, frameworks, or
application code.

Do not add a custom scheduler, database, dashboard, or GitHub App in the base
template.

Do not make human ergonomics the primary design target. Human readability matters
only insofar as it helps maintainers supervise and agents recover.

