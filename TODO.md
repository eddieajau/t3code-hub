# TODO

## Next Steps

### Task 3: Dev environment setup

Update INSTALL.md so a fresh clone can build and run t3code.

- [x] Add bun install via Homebrew
- [x] Add bun install step for t3code dependencies
- [x] Add instructions to install and pull an Ollama model
- [x] Verify: fresh terminal, follow INSTALL.md, `bun run typecheck` passes in `../t3code`

### Task 4: Contract changes — extend ProviderKind for Ollama

All code changes on branch `ollama-adapter` (create if not exists) in `../t3code`.

- [x] Run `bun run typecheck` and `bun run test` baseline and note any pre-existing failures
- [x] Add `"ollama"` to `ProviderKind` in `packages/contracts/src/orchestration.ts`
- [x] Add `OllamaProviderStartOptions` to `ProviderStartOptions` in `orchestration.ts` and `provider.ts`
- [x] Add `ollama` entries to all 5 `Record<ProviderKind, ...>` objects in `model.ts`
- [x] Run `bun run typecheck` — passes (expect errors in server code until adapter exists — contracts package itself must be clean)
- [x] Run `bun run test` — no new failures compared to baseline

### Task 5: OllamaAdapter service definition

- [x] Run `bun run typecheck` and `bun run test` baseline and note any pre-existing failures
- [x] Create `apps/server/src/provider/Services/OllamaAdapter.ts` mirroring `CodexAdapter.ts` (use `import * as Ollama` for ollama-specific types)
- [x] Run `bun run typecheck` — passes for the services file (no layer yet)
- [x] Run `bun run test` — no new failures compared to baseline

### Task 6: OllamaAdapter layer — session lifecycle (no chat yet)

Implement startSession, stopSession, listSessions, hasSession, stopAll. No sendTurn yet — just get the skeleton compiling and wired up.

- [x] Run `bun run typecheck` and `bun run test` baseline and note any pre-existing failures
- [x] Create `apps/server/src/provider/Layers/OllamaAdapter.ts` with session map + event queue
- [x] Implement `startSession` — create session state, emit `session.started` + `session.state.changed`
- [x] Implement `stopSession`, `listSessions`, `hasSession`, `stopAll`
- [x] Stub remaining methods (`sendTurn`, `interruptTurn`, `readThread`, `rollbackThread`, `respondToRequest`, `respondToUserInput`)
- [x] Run `bun run typecheck` — passes
- [x] Run `bun run test` — no new failures compared to baseline

### Task 7: Wire OllamaAdapter into the server

- [x] Run `bun run typecheck` and `bun run test` baseline and note any pre-existing failures
- [x] Add `ollamaBaseUrl` to `ServerConfigShape` in `config.ts`
- [x] Read `T3CODE_OLLAMA_BASE_URL` env var in `main.ts` (default `http://localhost:11434`)
- [x] Import and provide `OllamaAdapter` in `ProviderAdapterRegistry.ts`
- [x] Create `ollamaAdapterLayer` in `serverLayers.ts` and provide alongside codex
- [x] Run `bun run typecheck` — passes
- [x] Run `bun run test` — no new failures compared to baseline

### Task 8: OllamaAdapter layer — sendTurn with streaming

- [x] Run `bun run typecheck` and `bun run test` baseline and note any pre-existing failures
- [x] Implement `sendTurn` — append user message, emit turn events, POST to `/api/chat`, stream NDJSON
- [x] Each chunk emits `content.delta` with `streamKind: "assistant_text"`
- [x] On completion: append assistant message to history, emit `turn.completed` + `session.state.changed`
- [x] On error: emit `runtime.error`
- [x] Implement `interruptTurn` — abort the fetch via AbortController
- [x] Run `bun run typecheck` — passes
- [x] Run `bun run test` — no new failures compared to baseline

### Task 9: Manual end-to-end test

- [x] Verify: start Ollama locally (`ollama serve` + `ollama pull llama3:8b`)
- [x] Verify: start t3code server with `T3CODE_OLLAMA_BASE_URL=http://localhost:11434`
- [x] Verify: create a thread with `provider: "ollama"`, send a message, see streaming response in UI

### Task 10: Spike write-up and findings

- [ ] Update `docs/spike-ollama.md` with findings, what worked, what didn't
- [ ] Document tool-use gap analysis
- [ ] Assess whether a PR to upstream is viable
