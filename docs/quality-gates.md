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

The audit agent does not edit the branch. It reports pass or fail.

## Audit Output

Audit output should be written to the PR and linked issue:

```md
## Audit Result

Result: pass | fail

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

For a minimal GitHub-native setup, the audit signal can be represented by a
label, status check, or required PR review. The repository should choose the
simplest enforceable mechanism available.

## Failed Audit

When audit fails:

1. The auditor records concrete findings.
2. The auditor wakes the coordinator.
3. The coordinator returns the task to the original implementer.
4. The implementer fixes the PR and repeats CI.
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

