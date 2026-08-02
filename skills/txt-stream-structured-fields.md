---
name: Stream structured output field-by-field (JSON Patch)
description: >-
  Use dottxt's stream:"patch" mode to receive a schema-constrained response as
  RFC 6902 add operations, so routing/classification fields can trigger
  downstream work while long-form fields are still generating.
api: openapi/txt-dottxt-openapi-original.json
operations: [chat_completions]
---

# Stream structured output field-by-field (JSON Patch)

## Auth

`Authorization: Bearer <DOTTXT_API_KEY>` against `https://api.dottxt.ai/v1`.

## Steps

1. **Order the schema by priority.** Fields stream in schema order — put
   routing keys (intents, classifications, gates) first and long-form fields
   (replies, explanations) last.
2. **Call `chat_completions`** (`POST /chat/completions`) with
   `stream: "patch"` and a `response_format` JSON schema (required for patch
   mode). Everything else (`messages`, `temperature`, `max_tokens`, `seed`)
   behaves as in a normal call.
3. **Consume the stream.** Default framing is NDJSON
   (`application/x-ndjson`), one RFC 6902 `add` op per line; send
   `Accept: text/event-stream` for SSE framing (adds a final `event: done`).
   The first op seeds the root (`path: ""`, `value: {}` or `[]`); leaf fields
   arrive as `/field`, nested items as `/steps/0`, `/address/city`.
4. **Act on fields as they land** (dispatch on the JSON Pointer), and/or apply
   ops in order to reconstruct the full document — the result equals the
   non-streaming response. The Python SDK (`pip install dottxt`,
   `AsyncDotTxt.stream(...)`) yields `PatchEvent` objects with `field`,
   `value`, and a running `snapshot`.

## Rules

- Validation/auth failures return a standard JSON error before any patch
  records; a non-200 status means no patch records will arrive.
- Mid-stream connection failures surface as a closed stream without a
  terminator (Python SDK: `dottxt.PatchStreamError`) — reconcile by re-running
  the request; there is no resume cursor.
