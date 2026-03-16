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

- [ ] Review provider abstraction findings from Task 1
- [ ] Design OllamaAdapter implementing the provider interface
- [ ] Map Ollama chat completion API to t3code's event stream
- [ ] Handle tool-use capabilities (or fall back to chat-only mode)
- [ ] Validate with local Ollama instance running a code-capable model
