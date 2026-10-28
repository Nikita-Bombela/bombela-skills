---
name: break
description: Use when the user wants to find where a specific part of the app breaks under load - out-of-memory risks, CPU-heavy paths, or too many / too complex database calls.
argument-hint: <path | module | route | feature>
disable-model-invocation: true
---

# Break

Find the weakest point of `$ARGUMENTS` under load. Report only, don't fix.

If no argument was given, ask which part of the app to probe and stop.

## 1. Scope

Resolve `$ARGUMENTS` to concrete entry points (routes, handlers, jobs, exported functions). Trace each one down through everything it calls, including the data layer. State the resolved scope in one line. If it is ambiguous, ask.

Static reading only. Never run load tests or heavy queries against shared or production systems; propose the repro instead.

## 2. Probe

For every entry point ask: what input size or traffic makes this fall over?

**Memory**

- whole result sets, files or payloads loaded at once (no pagination, no streaming)
- unbounded caches, maps, queues, buffers with no eviction
- per-request collections that grow with user-controlled input
- request bodies / uploads with no size limit
- listeners, timers, closures or connections never released

**CPU**

- nested loops over related collections (O(n²) and worse)
- regex on user input (catastrophic backtracking)
- heavy sync work on the request path or event loop (hashing, compression, serializing big objects, image work)
- recursion without a depth limit
- the same expensive computation repeated per item or per request

**Database**

- N+1: queries inside loops, ORM lazy loading in serializers
- queries without LIMIT, `SELECT *` on wide tables
- filters, joins or sorts on columns without indexes
- OFFSET pagination on large tables
- heavy joins / aggregations on hot paths
- transactions held open across slow work or external calls, lock contention
- connections not returned to the pool, no pool limit

**Bounds.** List what limits exist: timeouts, rate limits, page size caps, payload limits. A missing bound on user-controlled input is a finding.

## 3. Rank

For each finding, note:

- **trigger:** the input or traffic that breaks it
- **blast radius:** one request, the whole process, or the DB for everyone
- **reach:** any anonymous user, logged-in user, or admin only

Severity:

- **critical:** user-controlled input can take down the process or DB
- **high:** degrades for everyone at realistic scale
- **medium:** breaks only beyond current scale, or only hurts the caller
- **low:** inefficiency with no failure mode

## 4. Report

Weakest point first. One verdict line, then numbered findings:

```
weakest point: export endpoint loads every row into memory, one large tenant OOMs the worker.

1. [critical][memory] export builds the full CSV in memory: tenant with 2M rows, worker dies. src/export.ts:40
2. [high][db] order list runs one query per item: 500 items = 501 queries. src/orders/list.ts:88
3. [low][cpu] tags deduped with a nested loop, fine below ~10k tags. src/tags.ts:12
```

Don't list what's fine. "rest holds" is the whole line.

End with one next action: which finding to fix first, or the smallest repro that proves it (input size, script, query to run locally).
