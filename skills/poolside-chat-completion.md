---
name: Call a poolside Laguna model (OpenAI-compatible)
description: >-
  Authenticate to poolside and send an agentic-coding chat-completion request to a
  Laguna model over the OpenAI-compatible inference API, including model discovery
  and streaming.
api: Poolside API (OpenAI-compatible)
base_url: https://inference.poolside.ai/v1
operations:
  - GET /v1/models
  - POST /v1/chat/completions
method: generated
source: https://docs.poolside.ai/api/openai-api-examples
---

# Call a poolside Laguna model

poolside serves its Laguna agentic-coding models through an **OpenAI-compatible**
API. The OpenAI SDKs work unchanged — you only switch the base URL and API key.

## 1. Get an API key

Sign in at `https://platform.poolside.ai` (free developer access), use an
OpenRouter key, or get a token from your Poolside deployment admin. In CI, pass it
as the `POOLSIDE_API_KEY` environment variable. All requests use:

```
Authorization: Bearer <api-key>
```

## 2. Pick the base URL

- Poolside Platform: `https://inference.poolside.ai/v1`
- Self-managed deployment: `https://<api-domain>/openai/v1`
- OpenRouter: `https://openrouter.ai/api/v1`

## 3. Discover available models — `GET /v1/models`

```bash
curl https://inference.poolside.ai/v1/models \
  -H "Authorization: Bearer $POOLSIDE_API_KEY"
```

Use a returned id (e.g. `poolside/laguna-m.1`) as the `model` in the next step.

## 4. Send a chat completion — `POST /v1/chat/completions`

```bash
curl https://inference.poolside.ai/v1/chat/completions \
  -H "Authorization: Bearer $POOLSIDE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "poolside/laguna-m.1",
    "messages": [{"role": "user", "content": "Refactor this function..."}],
    "max_completion_tokens": 1024,
    "temperature": 0.2
  }'
```

Optional parameters: `stream` (SSE deltas), `tools` (function/tool calling),
`response_format` (JSON-schema structured output).

## Rules

- Requests are **not idempotent** and there is no idempotency-key contract — do
  not blindly retry a completed POST; retry only on transport/5xx failure with
  backoff (see `conventions/poolside-conventions.yml`).
- Errors return the OpenAI envelope `{"error": {...}}` — handle 401 (bad key),
  429 (rate limit, back off), 404 (unknown model) per `errors/poolside-problem-types.yml`.
- For long context, Laguna models support a 256K window; keep prompt + completion
  within it.
