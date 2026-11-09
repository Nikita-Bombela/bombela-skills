---
name: probe
description: Walk the user through how a functionality works, step by step.
argument-hint: <path | module | route | feature>
disable-model-invocation: true
---

# Probe

Walk the user through how `$ARGUMENTS` works, as one numbered list of steps in a single reply. The user asks follow-ups if they want more. Read only, don't change code.

If no argument was given, ask which part to walk through and stop.

## 1. Flow

- Find the entry point (function, route, handler, job) and trace its whole path from input to output. If `$ARGUMENTS` matches several entry points, ask.
- Separate the functionality into small steps (a step is defined below). E.g. verify user is logged in, get arguments from the payload, save process key into Redis.
- Number the main path 1, 2, 3. A branch off step 3 is 3.1, 3.2: one line each, saying where it ends (returns 401, retries, rejoins at step 5).

## 2. Step

- A step is the minimal logical part without which the process would not work or would not be the same. Logging and metrics aren't steps.
- Steps shouldn't be too granular.
  - Bad: slice from 7th character to end on string meow_key
  - Good: Retrieve salesforce id from passed key
- Steps should be described in plain language, without preamble or summary.
- End each step with its location: `src/auth/session.ts:42`.
- Your own code that lives elsewhere (another repo, a Lambda, a queue consumer): follow it if you can read it, otherwise end the step with `[uncertain: <why>]`.
- Third-party libraries and SDKs: name the call, don't read their source.
  - Good: Uses the AWS SDK to get files from S3

## 3. Reply

The walkthrough is the body of your reply, in the same markdown as any other answer: a bold line naming the entry point, then the numbered steps, one line each. Branches are nested bullets under their step. Each line is `<what happens>. <file:line>`, then `[uncertain: <why>]` if it applies. No intro or summary around the list. Example reply:

**Entry:** `POST /sync`, `src/routes/sync.ts:18`

1. Verify the user is logged in. `src/middleware/auth.ts:12`
   - 1.1 No session or it expired: returns 401. `src/middleware/auth.ts:20`
2. Get the record key and options from the payload. `src/routes/sync.ts:22`
   - 2.1 Key missing: returns 400. `src/routes/sync.ts:25`
3. Retrieve the Salesforce id from the key. `src/sync/keys.ts:8`
4. Save the process key into Redis with a 10 min TTL. `src/sync/lock.ts:30`
   - 4.1 Key already there, another sync is running: returns 409. `src/sync/lock.ts:34`
5. Fetch the record from Salesforce with jsforce. `src/sync/salesforce.ts:41`
   - 5.1 Record not found: rejoins at step 8 with `status: missing`. `src/sync/salesforce.ts:47`
6. Upload the record as JSON to S3 with the AWS SDK. `src/sync/storage.ts:17`
7. Publish a `sync.done` event, billing-service picks it up. `src/sync/events.ts:9` [uncertain: billing-service isn't in this repo or reachable]
8. Return 202 with the process key. `src/routes/sync.ts:35`
