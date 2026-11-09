---
name: dry-new-code
description: Use this to check for repeats and redundancies in the new code. Goal of the skill is to prevent same functionality from emerging in new places and to maintain a single source of truth.
disable-model-invocation: true
---

# dry-new-code

Review everything this branch changes against the base branch (`$ARGUMENTS`, or `main` if empty). Report findings in chat only, don't fix anything.

## 1. Scope

- `git diff --name-status $(git merge-base <base> HEAD)` lists every file the branch touched, committed or not. `git status --short` adds untracked files.
- Skip lockfiles and generated, vendored or binary files.
- Read each changed file in full, not just the changed lines. For changed functions, signatures and schemas, also read their callers: bugs live where new code meets old.
- State the scope in one line: base, commits, files reviewed. If nothing changed, say so and stop.

## 2. Check

- For every new constant, function, class and type, search the rest of the repo before judging it, not just the diff:
  - its name and likely synonyms
  - its literal value (constants) or distinctive lines from its body (functions)
  - shared homes: `utils/`, `helpers/`, `lib/`, constants files, shared types
  - the standard library and installed dependencies
- For every new constant created, check if a constant with the same value already exists. If it does, check if it can be reused. If unsure, surface to the user.
- For every new function and class, check if it can either be integrated cleanly into existing functionality or already exists in a very similar form.
- Check if directly change something that used to be only changed from a single place. Preserve single source of truth when possible.
- Avoid hasty abstractions. If extending a functionality of an existing class or function requires too many arguments, inner juggling and internal checks, avoid it.

## 3. Feedback

Each finding names both locations: the new code and the existing source of truth.

```
1. [confidence: high] formatPrice() duplicates money.format().
   new: src/cart/total.ts:14 · existing: src/lib/money.ts:8
   Reuse: import from lib/money, drop the local copy.
```

- Confidence: high (same meaning, both sides read), medium (looks the same, depends on something not checked), low (similar shape, not confirmed).
- State what you find with code examples and provide the reason why you want to DRY it.
- If in doubt, better to report than avoid. Mark it low confidence.
- Don't list what's fine. If nothing is found, say so in one line and stop.
