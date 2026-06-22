# Quality Gates

Quality gates prevent incomplete or low-quality agent output from reaching
`main`.

The workflow uses two gate families:

- CI gates for deterministic checks.
- Agent audit gates for intent and quality review.

## CI Gate

CI should check what can be checked mechanically:

- build,
- tests,
- lint,
- formatting,
- type checks,
- docs checks,
- security scans when appropriate,
- repository template consistency checks.

The implementer owns CI until it passes. The coordinator should not launch audit
for a PR whose required CI checks are failing.

## Audit Gate

The audit agent checks whether the PR should be accepted. It reviews:

- the linked issue,
- acceptance criteria,
- diff,
- CI results,
- tests added or omitted,
- documentation changes,
- risk and rollback notes,
- whether the implementer took shortcuts,
- whether the solution is smaller or larger than necessary,
- whether the PR violates project rules.

The audit agent does not edit the branch. It records its findings as a GitHub
PR review, using inline comments for specific code issues and a formal review
outcome for the overall decision.

## Audit Output

Audit output should be written to the PR and linked issue:

```md
## Audit Result

GitHub review: APPROVE | REQUEST_CHANGES | COMMENT

## Evidence

- Reviewed issue: #123
- Reviewed PR: #456
- CI status: passed
- Important files reviewed:

## Findings

## Required Fixes

## Residual Risk
```

## Branch Protection

The protected `main` branch should require:

- required CI checks,
- required audit signal,
- PR review or agent audit marker,
- no direct pushes,
- no force pushes.

For a minimal GitHub-native setup, the audit signal should preferably be the
native PR review state itself. A label or status check can be added later if the
branch protection model needs an extra machine-readable gate.

## Failed Audit

When audit fails:

1. The auditor submits a `REQUEST_CHANGES` review with concrete findings.
2. The coordinator reads the PR review state and marks the task as returning to
   implementation.
3. The original implementer reads the GitHub review comments and fixes the PR.
4. The implementer repeats CI.
5. The coordinator launches a fresh audit after CI passes again.

## Human Escalation

Escalate to a human when:

- requirements conflict,
- the implementation requires product judgment,
- security risk is unclear,
- CI cannot represent the risk,
- agents disagree repeatedly,
- external credentials or permissions are needed,
- branch protection or repository settings must change.
