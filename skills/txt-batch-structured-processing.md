---
name: Run bulk structured generation as a batch
description: >-
  Upload a JSONL file of chat/embeddings requests, estimate its cost, run it
  as an asynchronous batch on the dottxt API, monitor progress, retry
  failures, and collect results.
api: openapi/txt-dottxt-openapi-original.json
operations: [upload_file, get_file_cost_estimate, create_batch, get_batch, get_batch_results, retry_failed_batch_requests, get_batch_analytics]
---

# Run bulk structured generation as a batch

## Auth

`Authorization: Bearer <DOTTXT_API_KEY>` against `https://api.dottxt.ai/v1`.

## Steps

1. **Build the JSONL file.** One JSON object per line with `custom_id` (your
   tracking id), `method` (POST), `url` (e.g. `/v1/chat/completions`), and
   `body` (the request payload, including `response_format` for structured
   output).
2. **Upload it with `upload_file`** (`POST /files`). Watch for 413 (file too
   large) and 403 (upload forbidden).
3. **Optionally price it with `get_file_cost_estimate`**
   (`GET /files/{file_id}/cost-estimate`) before spending credits.
4. **Start the run with `create_batch`** (`POST /batches`) referencing the
   uploaded file; choose a 1-hour or 24-hour completion window. Processing
   begins immediately (201).
5. **Monitor with `get_batch`** (`GET /batches/{batch_id}`) until the run
   completes; `get_batch_analytics` adds per-run analytics
   (`include=analytics` also works on list calls).
6. **Collect output with `get_batch_results`**
   (`GET /batches/{batch_id}/results`), matching lines back to your
   `custom_id`s.
7. **Retry failures with `retry_failed_batch_requests`**
   (`POST /batches/{batch_id}/retry`) — or retry selected `custom_id`s via
   `retry_specific_requests`.

## Rules

- Listings (`list_batches`, `list_files`) use cursor pagination: pass the
  response `last_id` as `after`, `limit` max 100, stop when `has_more` is
  false (`conventions/txt-conventions.yml`).
- There are no idempotency keys: guard against double-submitting a batch by
  checking `list_batches` for an existing run over the same input file before
  creating a new one.
- Errors use the OpenAI envelope (`errors/txt-problem-types.yml`); cancel an
  errant run with `cancel_batch`.
