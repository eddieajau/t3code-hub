# Spike Retrospective: Ollama Provider Adapter

**Date:** 2026-03-16
**Branch:** `ollama-adapter` in `../t3code`
**Status:** Spike complete. Not recommended for upstream PR at this time.

## What we built

A chat-only Ollama provider adapter for t3code, enabling local LLM
conversations through the existing UI. The adapter streams responses
from Ollama's `/api/chat` endpoint and integrates with the full
provider lifecycle (session start/stop, turn streaming, interrupt).

### Scope of changes

| Area | Files | Summary |
|------|-------|---------|
| Contracts | `ollama.ts`, `model.ts`, `provider.ts`, `orchestration.ts` | New provider kind, model catalog, start options |
| Adapter service | `provider/Services/OllamaAdapter.ts` | Service tag + shape definition |
| Adapter layer | `provider/Layers/OllamaAdapter.ts` (477 lines) | Session lifecycle, streaming sendTurn, interruptTurn |
| Server wiring | `config.ts`, `main.ts`, `serverLayers.ts`, `ProviderAdapterRegistry.ts` | Config, env var, layer composition |
| Provider routing | `ProviderCommandReactor.ts`, `ProviderSessionDirectory.ts` | Model-to-provider resolution, accept "ollama" in persistence |
| Web UI | `session-logic.ts`, `ProviderModelPicker.tsx`, `Icons.tsx`, `ChatView.logic.ts`, `_chat.settings.tsx` | Dropdown entry, icon, model options |

6 commits, ~850 lines added.

## What worked

- **Design-first approach paid off.** The spike doc (`spike-ollama.md`)
  defined the architecture before any code was written. Implementation
  closely followed the plan with minimal deviations.
- **Mirroring CodexAdapter patterns.** Using the same Effect service/layer
  structure, event queue pattern, and session state shape meant the adapter
  slotted into the existing provider framework cleanly.
- **Native fetch + NDJSON streaming.** No new dependencies needed. The
  streaming implementation is straightforward (~50 lines for the core loop).
- **AbortController for interrupt.** Clean cancellation that maps naturally
  to the existing `interruptTurn` contract.

## What didn't work / surprises

### Provider routing was harder than expected

The biggest surprise was that adding a new provider to the contracts layer
didn't automatically route threads to the correct adapter. Three separate
issues had to be fixed:

1. **`ProviderSessionDirectory.decodeProviderKind()`** was hardcoded to only
   accept `"codex"`. Any other provider string was treated as a persistence
   error. This is a blocker for ANY new provider, not just Ollama.

2. **No model-to-provider resolution existed.** The server had no concept of
   "model X belongs to provider Y". When the UI sent a turn-start command
   with `model: "llama3:8b"`, the server defaulted to codex because no
   explicit provider mapping existed.

3. **UI draft thread provider not propagating.** The web UI's composer draft
   store wasn't reliably carrying the selected provider through to the
   turn-start command for new threads. The workaround was to prioritize
   model-based provider resolution on the server side.

These issues suggest the upstream codebase was designed for a single provider
and would need non-trivial routing changes to support multiple providers
cleanly.

### UI provider list is a separate whitelist

The provider dropdown is not driven by contracts. It has its own hardcoded
list (`PROVIDER_OPTIONS` in `session-logic.ts`) plus icon mappings and model
option records in multiple files. Adding a provider requires touching 5+ web
files. This would benefit from being data-driven.

### Test infrastructure gaps

Three categories of type errors remain in test files:

- `LegacyProviderRuntimeEvent` type only accepts `"codex"` as provider
- `ProviderAdapterRegistry` test layer doesn't provide `OllamaAdapter`
- `wsServer` test mock missing `ollamaBaseUrl` config field

These are test-only issues that don't affect runtime, but they indicate the
test harness assumes a single-provider world.

## Tool-use gap analysis

The adapter deliberately stubs these methods (they throw "not implemented"):

| Method | Purpose | Difficulty to implement |
|--------|---------|----------------------|
| `respondToRequest` | Handle approval requests (file changes, commands) | **Hard** — requires adapter-side tool execution loop |
| `respondToUserInput` | Handle structured user input prompts | **Medium** — needs UI integration |
| `readThread` | Return thread conversation state | **Easy** — return in-memory messages |
| `rollbackThread` | Undo N turns from history | **Easy** — splice message array |

The critical gap is **tool calling**. Ollama supports tool definitions in the
chat API, but t3code's orchestration layer expects the provider to:

1. Receive tool definitions from the orchestration layer
2. Execute tool calls (file reads, command execution, patches)
3. Send approval requests back through the event system
4. Wait for user approval before proceeding

This is fundamentally different from Ollama's tool-calling model (where the
client is responsible for executing tools and sending results back). Bridging
this gap would require either:

- An adapter-side tool execution loop (essentially reimplementing what Codex
  does natively), or
- Upstream changes to the orchestration layer to support a "client-executes-
  tools" model

Neither is trivial. This is the main reason to NOT pursue an upstream PR yet.

## Questions for upstream maintainers

Before considering a PR, these questions need answers:

1. **Is multi-provider a goal?** The routing infrastructure assumes a single
   provider. Is there appetite to make this properly pluggable?
2. **How should model-to-provider mapping work?** Our catalog-based approach
   works but feels brittle. Should providers register their models dynamically?
3. **What's the plan for the "coming soon" providers** (Claude Code, Cursor,
   OpenCode, Gemini)? Will they follow the same adapter pattern or is a
   different architecture planned?
4. **Is the `LegacyProviderRuntimeEvent` type intentionally locked to codex?**
   If so, is there a migration path for new providers?
5. **Tool execution model:** Would upstream accept a provider that doesn't
   support tools, or is tool support a hard requirement?

## Recommendation

**Do not submit an upstream PR at this time.**

The spike successfully proves that Ollama can work as a t3code provider for
basic chat. However, the number of routing workarounds needed, the lack of
tool support, and the single-provider assumptions baked into the codebase
make this premature for upstream.

**Suggested next steps:**

- Open a discussion issue on the t3code repo asking about multi-provider
  plans and whether community adapter contributions are welcome
- Keep the `ollama-adapter` branch as a local fork for personal use
- Monitor upstream for provider abstraction improvements that would make
  a clean integration possible
