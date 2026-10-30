---
name: ship-check
description: Use when the user is finishing a feature branch and wants everything it changes reviewed against the base branch for bugs, security, production readiness and architecture, with findings written to a review file.
argument-hint: [base branch, default main]
disable-model-invocation: true
---

# Ship-check

Review everything this branch changes against the base branch (`$ARGUMENTS`, or `main` if empty). Write findings to a new review file. Report only, don't fix.

## 1. Scope

- `git diff --name-status $(git merge-base <base> HEAD)` lists every file the branch touched, committed or not. `git status --short` adds untracked files.
- Skip lockfiles and generated, vendored or binary files.
- Read each changed file in full, not just the changed lines. For changed functions, signatures and schemas, also read their callers: bugs live where new code meets old.
- State the scope in one line: base, commits, files reviewed. If nothing changed, say so and stop.

## 2. Check

**Bugs**

- null, empty or huge inputs, off-by-one, wrong comparisons
- swallowed errors, missing awaits, partial writes with no rollback
- races: shared state, check-then-act, retries that aren't idempotent
- changed contracts (signatures, return shapes, schemas) whose callers weren't updated

**Security**

- new routes, handlers or jobs missing authn / authz
- user input reaching SQL, shell, file paths, HTML or redirects unescaped
- secrets in the diff, PII in logs
- new dependencies: needed, maintained, pinned

**Production readiness**

- debug leftovers: console.log, commented-out code, TODOs, hardcoded URLs or credentials
- config that belongs in env, missing defaults
- migrations: reversible, safe on a live table, safe while old code is still running
- external calls without timeouts, errors that never reach logs or monitoring
- new logic without a test

**Architecture**

Judge against the patterns already in this repo, not taste.

- logic in the wrong layer
- re-implements a helper that already exists
- abstraction with one use, coupling that makes the feature hard to change or remove

## 3. Rank

Severity:

- **critical:** blocks merge: security hole, data loss, broken main flow
- **high:** fix before merge: a bug users will hit, missing prod safety
- **medium:** fix soon: edge case, debt with a real cost
- **low:** nit

Confidence: high (traced end to end), medium (depends on something outside the diff), low (pattern is there, not confirmed).

## 4. Review file

Write a new file at the repo root named `REVIEW-<branch>-<timestamp>.md`: slashes in the branch name become `-` (short commit hash if HEAD is detached), timestamp from `date +%Y%m%d-%H%M`. Never overwrite an existing review. Example `REVIEW-feature-export-20261030-1415.md`:

```
# Review: feature/export vs main

**Verdict:** not ready. New export route leaks data across tenants.

1. [critical][security][confidence: high] /export skips the tenant check, any user reads any tenant's data. src/routes/export.ts:22
   Fix: add requireTenant like src/routes/orders.ts:10

2. [medium][prod][confidence: medium] payment client has no timeout, a slow provider hangs the request. src/pay/client.ts:15
   Fix: pass timeout: 5000

Rest holds.
```

Don't list what's fine. The reply is the verdict line, a count per severity, and the path to the review file. Don't paste the findings again.
