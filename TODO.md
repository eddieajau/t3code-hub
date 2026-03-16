# TODO

## Next Steps

### Task 3: Dev environment setup

Update INSTALL.md so a fresh clone can build and run t3code.

- [x] Add bun install via Homebrew
- [x] Add bun install step for t3code dependencies
- [x] Add instructions to install and pull an Ollama model
- [x] Verify: fresh terminal, follow INSTALL.md, `bun run typecheck` passes in `../t3code`

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
