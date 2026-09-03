---
name: debug-from-langfuse-trace
description: Query Langfuse traces, observations and media through its public API, and debug a production error from a trace — correlate it with the code, reproduce it locally, and write a fix plan. Use this whenever the user mentions Langfuse or a trace ID, asks why something failed in production, or wants help debugging an LLM/agent pipeline error, even if they don't name Langfuse explicitly.
---

# Debug from a Langfuse trace

Two parts: a reference for talking to the Langfuse API, and a debugging workflow that uses it. This skill assumes nothing about the project it runs in — ask the user for anything project-specific rather than guessing.

## Setup guard

1. Load credentials: if a `.env` exists at the repository root, source it. Then check that `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY` and `LANGFUSE_HOST` are set (`printenv | cut -d= -f1`). If any is missing → **STOP**: "Missing Langfuse credentials. Please set LANGFUSE_PUBLIC_KEY, LANGFUSE_SECRET_KEY and LANGFUSE_HOST."
2. Check reachability:
   ```bash
   curl -sf -u "$LANGFUSE_PUBLIC_KEY:$LANGFUSE_SECRET_KEY" \
     "$LANGFUSE_HOST/api/public/projects" | jq '.data[].name'
   ```
   On failure → **STOP**: "Unable to connect to Langfuse: `<ERROR>`. Check the credentials and host, and any VPN the instance requires."

## Langfuse API reference

Every endpoint uses basic auth with the key pair; pipe responses through `jq` to keep output readable.

- `GET /api/public/observations` — list observations. Params: `page`, `limit`, `name`, `type` (SPAN, GENERATION, EVENT), `level` (DEBUG, DEFAULT, WARNING, ERROR), `traceId`, `parentObservationId`, `userId`, `fromStartTime`, `toStartTime`, `environment`, `version`.
- `GET /api/public/traces` — list traces, same filters where they apply.
- `GET /api/public/traces/{traceId}` — a single trace with its full observation tree. No filters.
- `GET /api/public/traces/{traceId}/media` and `GET /api/public/media/{mediaId}` — attachments; both return presigned download URLs valid roughly an hour.

Attachments (PDF pages, images, audio) also appear inline in an observation's input or output as `@@@langfuseMedia:type=<mime>|id=<MEDIA_ID>|source=base64_data_uri@@@`. Pull out `<MEDIA_ID>` and fetch it from the media endpoint. To work out what a given attachment represents, walk up the observation's parent chain — parents are usually named after the resource being processed.

Trace and observation names come from the instrumentation in the code (`@observe()` decorators, dynamic renaming via `update_current_span` / `update_current_trace`, or manual spans), so a name is almost always greppable: either as a function name, or as the format string that built it.

## Workflow

Copy this checklist and track progress:

- [ ] Step 1: Confirm scope with the user
- [ ] Step 2: List candidate error observations
- [ ] Step 3: Triage and pick what to debug
- [ ] Step 4: Pull the full traces
- [ ] Step 5: Static analysis of the code
- [ ] Step 6: Reproduce locally
- [ ] Step 7: Write the fix plan

### Step 1: Confirm scope

Unless the user already said, use `AskUserQuestion` to settle: which environment, what time window, and which components are in play — the service that raised the error, plus any caller involved in the flow (gateway or backend-for-frontend, frontend, worker, cron). Ask for the paths of repositories outside the current one. If a path isn't accessible, ask once more; if it still isn't available, say you'll proceed with the service code alone and flag that as a blind spot.

Only the repository the user asked you to fix should be modified. Treat the others as read-only context.

### Step 2: List candidate error observations

Announce the query parameters to the user before sending the request, then:

```bash
curl -s -u "$LANGFUSE_PUBLIC_KEY:$LANGFUSE_SECRET_KEY" \
  "$LANGFUSE_HOST/api/public/observations?level=ERROR&limit=50&environment=<ENV>&fromStartTime=$(date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ)" \
  | jq
```

(BSD/macOS: `date -u -v-7d +%Y-%m-%dT%H:%M:%SZ`.) Adjust the window, limit or `name` filter if the result set is too thin or too noisy.

### Step 3: Triage and pick what to debug

For each error, extract the trace name, error message and level, the input/output of the failing observation, and its metadata (session ID, user ID, tags — tags often mark expected failures such as user cancellation). Group by message plus trace name to separate one-offs from recurring patterns. Share a table:

| # | Trace name | Error message | Count | Trace ID | Last seen |
|---|-----------|---------------|-------|----------|-----------|

Then use `AskUserQuestion` with a recommendation of which error(s) to take on this session — highest frequency and clearest signal first — and let the user choose.

### Step 4: Pull the full traces

Fetch `GET /api/public/traces/{traceId}` for each selected error and read the whole observation tree, not just the failing node: the cause often sits in a parent's input, a sibling's timing, or a retry that quietly exhausted. Fetch media attachments when the failure involves a document or image.

### Step 5: Static analysis of the code

1. **Map trace → code**: grep the observation name to find the instrumented function.
2. **Read the failing path** top-down through the observation hierarchy, reading each function involved.
3. **Grep the error message itself** — it often lands on the exact raise or log site.
4. **Check recent changes** on that path: `git log --oneline -20 -- <file>`, and `git log -S '<symbol>'` when a specific call looks suspicious.
5. **Check the usual suspects**: unhandled failures from external APIs; races or unbounded concurrency in async/parallel code; prompt and template problems (missing variables, remote prompt fetch failures); database session and connection handling; timeouts and retry exhaustion; and contract mismatches with the caller — what it actually sends versus what the service expects, and what the end user sees when it breaks.

### Step 6: Reproduce locally

Attempt reproduction whenever the failing path can run locally; skipping this step tends to produce incomplete fixes. If the project documents how to start and exercise it (README, Makefile, compose file, an e2e test suite or skill), use that. Otherwise ask the user how to run the service and how to replay a request.

Replay the failing scenario with the inputs from the trace — same parameters, same resource IDs, same feature flags. If those inputs point at production-only data, ask the user for an equivalent local fixture. Confirm the error reproduces (local logs and/or a fresh Langfuse trace), then re-run the same scenario after the fix and record the before/after.

### Step 7: Write the fix plan

Write the plan to `docs/error-reports/<descriptive-error-name>.md`, or to the project's own convention if it has one:

- **Root cause** — one or two sentences naming the file and function.
- **Proposed fix** — the concrete changes, plus anything deliberately left out of scope.
- **Testing strategy** — tests to add, and the reproduction scenario to re-run.
- **Estimated effort** — a rough size, to help prioritisation.

Share the path with the user and stop there unless they ask you to implement the fix.

