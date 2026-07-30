---
name: Converse with a Lua agent over the HTTP API
description: Send a message to a deployed Lua agent and read its response, either as a single completion or a streamed SSE reply.
api: https://api.heylua.ai
operations:
  - "POST /chat/generate/{agentId}"
  - "POST /chat/stream/{agentId}"
source: https://docs.heylua.ai/channels/http-api
generated: '2026-07-20'
method: generated
---

# Converse with a Lua agent

Use this skill to call a deployed Lua AI agent from your own code via the
public HTTP API at `https://api.heylua.ai`.

## Authenticate

- Obtain an API key with the CLI (`lua auth`, email OTP) or from the admin dashboard.
- Send it as a bearer token on every request: `Authorization: Bearer YOUR_API_KEY`.
- Keep the key in the `LUA_API_KEY` environment variable; never commit it.

## Single response — `POST /chat/generate/{agentId}`

1. POST JSON with a `messages` array of content parts (`{"type":"text","text":"..."}`).
2. Optional fields: `systemPrompt`, `runtimeContext`, `clientContext` (e.g.
   `clientContext.timezone` as an IANA tz string), `options`.
3. Optional query params: `channel` (context) and `identifier` (tracking).
4. Read the JSON reply: `{ "text": ..., "toolCalls": [...], "usage": {...} }`.

## Streamed response — `POST /chat/stream/{agentId}`

1. POST the same body to the stream endpoint.
2. Read Server-Sent Events: consume `{"type":"text-delta","textDelta":"..."}`
   chunks and stop on `{"type":"finish","finishReason":"stop"}`.

## Handle errors

- Errors return the envelope `{"type":"error","message":"...","statusCode":<int>}`.
- `400` bad request, `401` unauthorized (bad/missing key), `423` locked
  (retry when unlocked), `500` server error (retry with backoff).
- In the stream, an error arrives as an SSE chunk of the same shape.

## Notes

- No idempotency-key header is documented — do not assume safe automatic retries
  on non-idempotent turns.
- Grounded in the two real documented HTTP operations; the platform's richer
  capabilities are authored in-code with the `lua-cli` TypeScript SDK.
