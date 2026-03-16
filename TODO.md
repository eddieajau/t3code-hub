# TODO

## Next Steps

### Task 1: Thorough review of the t3code repo

**Deliverables:**

- `docs/t3code-review-YYYY-MM-DD.md` — Comprehensive review covering all areas below, with findings and recommendations per section
- `.claude/commands/` — Claude slash commands for upstream sync workflows (e.g. sync-check, upstream-diff)

**Pre-flight:** Sync the t3code fork with upstream before starting:

```bash
git -C ../t3code fetch upstream && git -C ../t3code merge upstream/main && git -C ../t3code push
```

Record the HEAD commit hash in the review doc.

**Checklist:**

- [x] Architecture & Design — event-sourcing model, Effect-TS value, package boundaries
- [x] Provider Abstraction — how pluggable is it really, how tightly is Codex wired in
- [x] Code Quality — duplication, dead code, pattern consistency, staleness of `.plans/` docs
- [x] Testing — coverage gaps, test quality, missing integration tests
- [x] DX (Developer Experience) — setup friction, debugging, onboarding
- [x] Governance Readiness — CONTRIBUTING.md, CI gates, PR process, docs for new contributors
- [x] Security & Operational Concerns — auth tokens, WebSocket security, process spawning, sandboxing
- [x] Recommendations — Claude commands and workflows for upstream sync (detecting stale research, diffing upstream changes, checking if planned PRs still make sense)

### Task 2: Spike — Ollama provider adapter for t3code

**Deliverables:**

- `docs/spike-ollama.md` — Spike write-up: findings, design decisions, what worked, what didn't, and whether a PR is viable
- Code changes in a t3code branch (prepared for PR when ready)

**Checklist:**

- [x] Review provider abstraction findings from Task 1
- [x] Design OllamaAdapter implementing the provider interface
- [x] Map Ollama chat completion API to t3code's event stream
- [x] Handle tool-use capabilities (or fall back to chat-only mode)
- [x] Validate with local Ollama instance running a code-capable model

### Task 3: Dev environment setup

Update INSTALL.md so a fresh clone can build and run t3code.

- [ ] Add bun install via Homebrew
- [ ] Add bun install step for t3code dependencies
- [ ] Add instructions to install and pull an Ollama model
- [ ] Verify: fresh terminal, follow INSTALL.md, `bun run typecheck` passes in `../t3code`

### Task 4: Contract changes — extend ProviderKind for Ollama

All code changes on branch `spike/ollama-adapter` in `../t3code`.

- [ ] Add `"ollama"` to `ProviderKind` in `packages/contracts/src/orchestration.ts`
- [ ] Add `OllamaProviderStartOptions` to `ProviderStartOptions` in `orchestration.ts` and `provider.ts`
- [ ] Add `ollama` entries to all 5 `Record<ProviderKind, ...>` objects in `model.ts`
- [ ] Verify: `bun run typecheck` passes (expect errors in server code until adapter exists — contracts package itself must be clean)

### Task 5: OllamaAdapter service definition

- [ ] Create `apps/server/src/provider/Services/OllamaAdapter.ts` mirroring `CodexAdapter.ts`
- [ ] Verify: `bun run typecheck` passes for the services file (no layer yet)

### Task 6: OllamaAdapter layer — session lifecycle (no chat yet)

Implement startSession, stopSession, listSessions, hasSession, stopAll. No sendTurn yet — just get the skeleton compiling and wired up.

- [ ] Create `apps/server/src/provider/Layers/OllamaAdapter.ts` with session map + event queue
- [ ] Implement `startSession` — create session state, emit `session.started` + `session.state.changed`
- [ ] Implement `stopSession`, `listSessions`, `hasSession`, `stopAll`
- [ ] Stub remaining methods (`sendTurn`, `interruptTurn`, `readThread`, `rollbackThread`, `respondToRequest`, `respondToUserInput`)
- [ ] Verify: `bun run typecheck` passes

### Task 7: Wire OllamaAdapter into the server

- [ ] Add `ollamaBaseUrl` to `ServerConfigShape` in `config.ts`
- [ ] Read `T3CODE_OLLAMA_BASE_URL` env var in `main.ts` (default `http://localhost:11434`)
- [ ] Import and provide `OllamaAdapter` in `ProviderAdapterRegistry.ts`
- [ ] Create `ollamaAdapterLayer` in `serverLayers.ts` and provide alongside codex
- [ ] Verify: `bun run typecheck` passes
- [ ] Verify: `bun run test` passes (existing tests unbroken)

### Task 8: OllamaAdapter layer — sendTurn with streaming

- [ ] Implement `sendTurn` — append user message, emit turn events, POST to `/api/chat`, stream NDJSON
- [ ] Each chunk emits `content.delta` with `streamKind: "assistant_text"`
- [ ] On completion: append assistant message to history, emit `turn.completed` + `session.state.changed`
- [ ] On error: emit `runtime.error`
- [ ] Implement `interruptTurn` — abort the fetch via AbortController
- [ ] Verify: `bun run typecheck` passes
- [ ] Verify: `bun run test` passes

### Task 9: Manual end-to-end test

- [ ] Verify: start Ollama locally (`ollama serve` + `ollama pull llama3.1`)
- [ ] Verify: start t3code server with `T3CODE_OLLAMA_BASE_URL=http://localhost:11434`
- [ ] Verify: create a thread with `provider: "ollama"`, send a message, see streaming response in UI

### Task 10: Spike write-up and findings

- [ ] Update `docs/spike-ollama.md` with findings, what worked, what didn't
- [ ] Document tool-use gap analysis
- [ ] Assess whether a PR to upstream is viable
