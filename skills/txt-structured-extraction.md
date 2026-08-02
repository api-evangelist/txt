---
name: Extract structured data with a schema contract
description: >-
  Call the dottxt API to turn unstructured text into schema-valid JSON. The
  JSON Schema you pass in response_format is compiled and enforced by
  constrained decoding, so the output always parses and always validates.
api: openapi/txt-dottxt-openapi-original.json
operations: [list_models, chat_completions]
---

# Extract structured data with a schema contract

## Auth

Send `Authorization: Bearer <DOTTXT_API_KEY>` (keys are prefixed `sk-dottxt-`)
against base URL `https://api.dottxt.ai/v1`. Verify the key first with
`list_models` (`GET /models`) — a 200 lists the models your key can use; pick
one of its `data[].id` values (e.g. `openai/gpt-oss-20b`).

## Steps

1. **Author the schema as a contract, not a hint.** Mark everything you need
   as `required`, set `additionalProperties: false`, and bound strings/arrays
   (`minLength`, `maxLength`, `minItems`, `maxItems`, `enum`, `pattern`).
   Check keyword support at https://docs.dottxt.ai/supported-features.
2. **Call `chat_completions`** (`POST /chat/completions`) with your prompt in
   `messages` and the schema wrapped as
   `response_format: {"type": "json_schema", "json_schema": {"name": "...", "schema": {...}}}`.
3. **Parse `choices[0].message.content`** — it is guaranteed to be valid JSON
   matching your schema; no retry-on-parse-failure loop is needed.

## Rules

- Errors use the OpenAI envelope `{"error": {code, message, param, type}}`
  (see `errors/txt-problem-types.yml`): 401 bad key, 402 insufficient
  credits, 403 model not allowed for this key, 404 unknown model, 429 back
  off and retry — no idempotency keys exist, so only retry reads and
  generation calls you are prepared to pay for twice.
- Conventions (pagination, streaming, error semantics):
  `conventions/txt-conventions.yml`.
