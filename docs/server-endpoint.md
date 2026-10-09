# Own AI server

MeetToKnow uses the **OpenAI Chat Completions API with streaming**. Works with llama.cpp (`llama-server`),
Ollama, LM Studio, `mlx_lm.server`, vLLM, OpenRouter, Groq, OpenAI.

## Settings ▸ AI ▸ Server

| Field | Value |
|---|---|
| Base URL | API root incl. `/v1` — `http://localhost:8080/v1` (llama.cpp), `http://localhost:11434/v1` (Ollama), `http://localhost:1234/v1` (LM Studio) |
| API key | optional, sent as `Authorization: Bearer <key>` |
| Model | via **Fetch models**, or typed |
| Context window | optional; empty = auto |

## Routes

### `GET {base}/models`

```json
{ "data": [ { "id": "gemma-4-E4B-it", "meta": { "n_ctx": 32768 } } ] }
```

- `data[].id` required
- `data[].meta.n_ctx` / `n_ctx_train` optional (context window)

### `POST {base}/chat/completions`

Headers: `Content-Type: application/json`, `Accept: text/event-stream`, optional `Authorization`. Timeout 300 s.

```json
{
  "model": "gemma-4-E4B-it",
  "messages": [{ "role": "system", "content": "…" }, { "role": "user", "content": "…" }],
  "stream": true,
  "stream_options": { "include_usage": true },
  "max_tokens": 2000,
  "temperature": 0.2
}
```

- Summaries: no tools.
- Chat: adds `tools` (one function: meeting search), then assistant messages with `tool_calls` and `role: "tool"` messages with `tool_call_id`. No tool support → summaries work, chat doesn't.

### Response: SSE

```text
data: {"choices":[{"delta":{"content":"Hello"}}]}
data: {"choices":[{"delta":{},"finish_reason":"stop"}],"usage":{"prompt_tokens":812,"completion_tokens":2}}
data: [DONE]
```

| Field | Use |
|---|---|
| `choices[0].delta.content` | answer text |
| `choices[0].delta.reasoning_content` | optional thinking, shown separately, not saved |
| `choices[0].delta.tool_calls[]` | chat: `id`, `function.name`, `function.arguments` (JSON string, may be split across chunks) |
| `choices[0].finish_reason` | `stop`, `length`, `tool_calls` |
| `usage` | optional token counts |

Errors: non-2xx status with a short text or JSON body.

## Context window

Order: Settings field → `meta.n_ctx` from `/models` → Ollama `POST {root}/api/show` → family default.
Set the field if your server is neither llama.cpp nor Ollama; a value above the real context breaks long summaries.

## Check

```bash
BASE=http://localhost:8080/v1
curl -s $BASE/models
curl -sN $BASE/chat/completions -H 'Content-Type: application/json' \
  -d '{"model":"YOUR-MODEL","stream":true,"stream_options":{"include_usage":true},"messages":[{"role":"user","content":"Say hello."}]}'
```

Expect `data:` lines with `delta.content`, ending in `data: [DONE]`.

## Prompt for building one

> Build an OpenAI-compatible server: `GET /v1/models` and streaming `POST /v1/chat/completions` (SSE, `data: [DONE]`, `stream_options.include_usage`), forwarding to <model>. Support `tools` / `tool_calls`.
