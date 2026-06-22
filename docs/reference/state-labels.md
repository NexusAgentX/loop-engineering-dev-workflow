# State Labels Reference

Labels are optional but useful for making agent state visible in GitHub.

The exact label names can be customized. The important part is that the
repository uses one consistent vocabulary.

## Issue State Labels

```text
state:new
state:triage
state:needs-info
state:ready
state:blocked
state:running
state:ready-for-audit
state:audit-running
state:audit-failed
state:audit-passed
state:merged
state:closed
state:cancelled
```

## PR State Labels

```text
pr:ci-wait
pr:ci-failed
pr:ci-passed
pr:audit-wait
pr:audit-running
pr:audit-failed
pr:audit-passed
pr:mergeable
pr:merged
pr:blocked
```

## Role Labels

```text
role:coordinator
role:implementer
role:auditor
```

## Workflow Labels

```text
workflow:simple
workflow:coordinated
workflow:root
workflow:child
workflow:parallel-safe
workflow:serialize
```

## Dependency Labels

```text
dependency:blocked
dependency:unblocked
dependency:root
dependency:child
```

## Recommended Minimum

A minimal repository can start with only these:

```text
state:ready
state:blocked
state:running
state:ready-for-audit
state:audit-failed
state:audit-passed
workflow:root
workflow:child
workflow:serialize
```

