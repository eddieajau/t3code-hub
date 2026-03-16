# Spike: Ollama Provider Adapter for t3code

**Date:** 2026-03-16
**Status:** Design complete, implementation pending

## Context

t3code currently only supports Codex as a provider. We want to add an Ollama adapter to enable local LLM usage. This is a spike — goal is a working chat-only adapter that follows upstream patterns closely. Tool support is deferred.

Ollama is stateless (full conversation history sent each request) and uses HTTP streaming (`POST /api/chat` with NDJSON response chunks). This is fundamentally simpler than Codex (which spawns a child process with JSON-RPC).

## Key Design Decisions

1. **Chat-only** — tool calling requires adapter-side tool execution loop, which conflicts with t3code's orchestration/approval flow. Deferred.
2. **No new npm dependencies** — use native `fetch` for HTTP. No `ollama` npm package.
3. **In-memory history** — acceptable for spike. Production would need persistence.
4. **No `RuntimeEventRawSource` changes** — omit `raw` field on events to avoid contract churn.
5. **Env var for base URL** — `T3CODE_OLLAMA_BASE_URL`, no CLI flag needed for spike.

## Architecture

### Contract Changes (`packages/contracts/`)

- `ProviderKind`: extend from `"codex"` to `"codex" | "ollama"`
- `ProviderStartOptions`: add optional `ollama` key with `baseUrl` field
- `model.ts`: add `ollama` entries to all 5 `Record<ProviderKind, ...>` objects (model options, defaults, aliases, reasoning effort)

### OllamaAdapter Service (`apps/server/src/provider/Services/`)

Mirror `CodexAdapter.ts` — a service tag + shape interface extending `ProviderAdapterShape<ProviderAdapterError>` with `provider: "ollama"`.

### OllamaAdapter Layer (`apps/server/src/provider/Layers/`)

Core patterns from CodexAdapter:
- `Queue.unbounded<ProviderRuntimeEvent>()` + `Stream.fromQueue()` for event stream
- `Effect.acquireRelease` for cleanup (queue shutdown)
- `Map<ThreadId, OllamaSessionState>` for session tracking
- Branded ID constructors: `EventId.makeUnsafe(...)`, `TurnId.makeUnsafe(...)`

Session state per thread:
```
OllamaSessionState {
  session: ProviderSession
  messages: Array<{ role, content }>  // full history
  turnCount: number
  currentAbortController: AbortController | null
}
```

sendTurn flow:
1. Append user message to history
2. Emit `turn.started`, `session.state.changed` (running)
3. Fork a fiber that streams `POST /api/chat` (NDJSON)
4. Each chunk -> `content.delta` event (streamKind: "assistant_text")
5. On done -> append assistant message to history, emit `turn.completed`, `session.state.changed` (ready)
6. On error -> `runtime.error`
7. Return `{ threadId, turnId }`

No-op methods (chat-only): `respondToRequest`, `respondToUserInput`

Capabilities: `{ sessionModelSwitch: "in-session" }` (Ollama is stateless, model can change per-request)

### Wiring

- `ProviderAdapterRegistry.ts`: import OllamaAdapter, add to default adapters
- `serverLayers.ts`: create ollamaAdapterLayer, provide alongside codex
- `config.ts`: add `ollamaBaseUrl` to ServerConfigShape
- `main.ts`: read `T3CODE_OLLAMA_BASE_URL` env var (default `http://localhost:11434`)

## Ollama API Reference

### POST /api/chat (streaming)

Request:
```json
{
  "model": "llama3.1",
  "messages": [
    { "role": "user", "content": "Hello" }
  ],
  "stream": true
}
```

Response (NDJSON):
```json
{"model":"llama3.1","message":{"role":"assistant","content":"Hello"},"done":false}
{"model":"llama3.1","message":{"role":"assistant","content":"!"},"done":false}
{"model":"llama3.1","message":{"role":"assistant","content":""},"done":true}
```

## Verification Plan

1. `bun run typecheck` passes
2. `bun run test` passes (existing tests unbroken)
3. Manual: start Ollama locally, start t3code server, create thread with `provider: "ollama"`, send a message, see streaming response in UI

## Findings

_To be filled in after implementation._

## Recommendation

_To be filled in after implementation — whether a PR to upstream is viable._
