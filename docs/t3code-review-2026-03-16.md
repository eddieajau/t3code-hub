# t3code Comprehensive Review

**Date:** 2026-03-16
**Upstream HEAD:** `4b811dae61b3042e2bdd053b52f9a75aac46fcb8`
**Reviewer:** Claude (automated analysis)

---

## 1. Architecture & Design

### Overview

t3code is a monorepo (Bun workspaces + Turbo) implementing a web GUI for code agents. Current structure:

- **Apps:** `server` (Node.js/Bun), `web` (React/Vite), `desktop` (Electron), `marketing` (Astro)
- **Packages:** `contracts` (Effect schemas only), `shared` (utilities with explicit subpath exports)

### Event-Sourcing Model

The server implements a **full event-sourcing + CQRS pattern**:

1. Client sends `ClientOrchestrationCommand` via WebSocket
2. `OrchestrationEngine` processes via unbounded queue (serial ordering)
3. `decideOrchestrationCommand()` produces events (decider pattern)
4. Events persist to SQLite `orchestration_events` table
5. `ProjectionPipeline` updates 9 denormalized projection tables
6. PubSub publishes to WebSocket subscribers

**Key files:**
- `apps/server/src/orchestration/decider.ts` — command→event logic
- `apps/server/src/orchestration/projector.ts` — event→read-model
- `apps/server/src/orchestration/Layers/OrchestrationEngine.ts` — core engine
- `apps/server/src/orchestration/Layers/ProjectionPipeline.ts` — 9 projectors (870 lines)
- `packages/contracts/src/orchestration.ts` — 21 event types, 2 aggregate kinds (project, thread)

**Assessment:** Production-grade implementation with full audit trail, replay capability, and clean separation of write/read paths.

### Effect-TS Usage

Effect-TS is used pervasively and consistently across the entire backend (~920+ occurrences):

- **Service/Layer pattern** for dependency injection (`makeServerRuntimeServicesLayer` composes 10+ subsystem layers)
- **Typed error channels** via `Effect.catchTag`, `Effect.mapError`
- **Concurrency primitives** — `Queue`, `PubSub`, `Stream` for event processing
- **Resource management** — `Scope`, `Layer.scoped` for WebSocket/DB connections
- **Schema validation** at all boundaries

**Assessment:** Effect-TS provides genuine value here — typed errors, composable services, and structured concurrency. Not a gratuitous adoption. Consistent usage across all server subsystems.

### Package Boundaries

Clean acyclic dependency graph:

```
contracts (zero internal deps — schemas only)
    ↑
shared (depends on contracts)
    ↑
server, web, desktop (depend on contracts + shared)
marketing (independent)
```

- No circular dependencies
- `contracts` has zero runtime logic (enforced by convention in AGENTS.md)
- `shared` uses explicit subpath exports — no barrel file bloat
- Each app is independent except for shared dependencies

**Assessment:** Boundaries are clean and well-enforced. The contracts-first approach ensures schema consistency across apps.

---

## 2. Provider Abstraction

### Interface Design

The provider system uses a clean adapter pattern:

- **Core interface:** `ProviderAdapterShape<TError>` (`apps/server/src/provider/Services/ProviderAdapter.ts:45-126`)
  - Methods: `startSession`, `sendTurn`, `interruptTurn`, `respondToRequest`, `respondToUserInput`, `stopSession`, `listSessions`, `hasSession`, `readThread`, `rollbackThread`, `stopAll`, `streamEvents`
  - Capabilities declaration (`sessionModelSwitch` mode)

- **Registry:** `ProviderAdapterRegistryShape` (`apps/server/src/provider/Layers/ProviderAdapterRegistry.ts:24-43`)
  - `getByProvider(kind)` — returns adapter by `ProviderKind`
  - `listProviders()` — lists available providers
  - Accepts custom adapters at construction time

- **Service facade:** `ProviderService` routes operations through the registry

### How Tightly is Codex Wired In?

**Moderately tight — abstraction exists but type-level lock limits extension:**

1. **`ProviderKind = Schema.Literal("codex")`** (`packages/contracts/src/orchestration.ts:30`) — only "codex" is a valid provider at compile time. This is the primary bottleneck for adding providers.

2. Registry defaults to `[yield* CodexAdapter]` but accepts custom adapters via options.

3. Event canonicalization (`mapToRuntimeEvents()` in `CodexAdapter.ts:519-1200+`) is heavily Codex-specific — 700+ lines mapping Codex native events to canonical `ProviderRuntimeEvent`.

### What It Takes to Add a New Provider

1. Extend `ProviderKind` union in contracts (e.g., add `"ollama"`)
2. Create adapter service definition implementing `ProviderAdapterShape`
3. Implement the adapter layer with event mapping to canonical `ProviderRuntimeEvent`
4. Register in `ProviderAdapterRegistry` and wire into `serverLayers.ts`

**Estimated effort:** 4-5 days for a non-Codex provider (biggest cost: event mapping + synthetic event generation for providers without native event streams).

**Assessment:** The abstraction is genuine and testable (integration tests use `TestProviderAdapter`). The `ProviderKind` literal type is the main friction point — extending it is a one-line change but signals this was designed for Codex first.

---

## 3. Code Quality

### Duplication

Several helper functions are duplicated across modules:

- **`asString()` / `asObject()`** — defined independently in 3 locations:
  - `apps/server/src/codexAppServerManager.ts:168-177`
  - `apps/server/src/provider/Layers/CodexAdapter.ts:93-102`
  - `apps/server/src/orchestration/Layers/ProviderRuntimeIngestion.ts:97-99`

- **`toTurnId()`** — 3 variants with different return types (null vs undefined)
  - `CheckpointReactor.ts:39-41`
  - `ProviderRuntimeIngestion.ts:55-57`
  - `codexAppServerManager.ts`

- **`sameId()`** — duplicated in `ProviderRuntimeIngestion.ts:63-68` and `CheckpointReactor.ts:43-48`

### Dead Code

- `latestMessageIdByTurnKey` map in `ProviderRuntimeIngestion.ts:133` — written to but never read (tracked as C046 in remediation checklist, marked DONE)
- Some error classes in `checkpointing/Errors.ts` may be unused (tracked as C033-C035, marked DONE)

### Large Files

| File | Lines | Concern |
|------|-------|---------|
| `codexAppServerManager.ts` | 1,587 | God object — manages sessions, JSON-RPC, account snapshots, plan modes |
| `CodexAdapter.ts` | 1,525 | Large but mostly event mapping — potentially splittable |
| `GitCore.ts` | 1,428 | Complex parsing logic |
| `composerDraftStore.ts` | 1,298 | Complex state with multiple record maps |
| `ProjectionPipeline.ts` | 1,232 | 9 projector definitions |
| `Manager.ts` (terminal) | 1,226 | 6+ internal Maps/Sets |
| `ProviderRuntimeIngestion.ts` | 1,147 | Cache-backed message tracking |

### Staleness of Planning Docs

Planning docs are **actively maintained**:
- `.plans/16c-pr89-remediation-checklist.md` — 51 actionable items, Phases 1-3 mostly DONE, Phases 4-6 still TODO
- `.docs/architecture.md` — accurately describes current 3-layer model
- `.docs/workspace-layout.md` — correct structure

### Outstanding Remediation Items (Phases 4-6)

- **C018**: `handledTurnStartKeys` Set grows unbounded on long-running server
- **C019**: Race condition in thread event routing
- **C050**: Read-modify-write race in `ProviderSessionDirectory.ts:94`
- **C054**: Multi-byte UTF-8 chunk splitting in `wsServer.ts:104`
- **C038**: Same UTF-8 issue in `CodexTextGeneration.ts:136`
- **C052**: Race condition — `processHandle` null when data callback fires in `BunPTY.ts:97`
- **C009**: Git braced rename syntax not handled in `GitCore.ts:41`
- **C010**: Missing keybindings config ENOENT not handled
- **C022/C023**: Fish shell PATH handling issues

**Assessment:** Code quality is solid for an early-stage project. The remediation checklist shows disciplined tracking of tech debt. Main concerns are the 7 large files (1,200+ lines each) and helper duplication that could be extracted to shared utilities.

---

## 4. Testing

### Framework & Configuration

- **Vitest 4.0.0** with `@effect/vitest` for Effect-TS integration
- **Playwright** for browser component tests (chromium)
- Root config + server-specific config (15s timeout) + web browser config (30s timeout)

### Coverage by Package

| Package | Source Files | Test Files | Coverage |
|---------|-------------|------------|----------|
| `apps/web` | 138 | 41 | ~30% |
| `apps/server` | 124 | 43 | ~35% |
| `apps/desktop` | 7 | 5 | ~71% |
| `packages/contracts` | 14 | 7 | ~50% |
| `packages/shared` | 7 | 4 | ~57% |
| `apps/marketing` | — | 0 | 0% |

**Total:** 102 test files, ~30,846 lines of test code.

### Test Types

- **Unit tests** — majority; schema validation, pure logic, state management
- **Integration tests** — 2 files in `apps/server/integration/` (orchestration engine, provider service)
- **Browser tests** — 2 component tests via Playwright (`ChatView`, `KeybindingsToast`)
- **No E2E tests** for full user flows

### Notable Gaps

- **Contracts package** missing tests for: `project.ts`, `editor.ts`, `model.ts`, `baseSchemas.ts`, `ipc.ts`, `server.ts`
- **Shared package** missing tests for: `schemaJson.ts`, `git.ts`, `logging.ts`
- **Marketing app** has zero tests
- Only 2 browser rendering tests — minimal UI regression coverage
- No performance benchmarks
- No visual regression testing

### Test Quality

Where tests exist, quality is strong:
- Effect-TS integration with proper resource management in integration tests
- Fixture-based testing with deterministic event streams
- Schema validation tests cover both positive and negative cases
- Complex temporal logic tested with multiple edge cases

**Assessment:** Testing is respectable for the project's stage (~30-35% file coverage in core apps, 30K LOC). The integration test harness is well-designed. Main gaps are untested contract schemas (which are critical for type safety), limited browser tests, and no E2E flows.

---

## 5. Developer Experience

### Setup Friction: LOW

- `bun install && npm run dev` gets you running
- Deterministic port allocation via `dev-runner.ts` (base ports: server 3773, web 5733 with hash offsets)
- No `.env.example` needed — environment managed programmatically
- Version pinning: Bun 1.3.9+, Node 24.13.1+

### Debugging: STRONG

- Source maps enabled by default (configurable via `T3CODE_WEB_SOURCEMAP`)
- WebSocket event logging via `T3CODE_LOG_WS_EVENTS`
- HMR configured for Electron compatibility
- Dev state directory defaults to `~/.t3/dev`

### Onboarding: CLEAR BUT RESTRICTIVE

- AGENTS.md defines package roles, priorities, and task completion requirements
- CONTRIBUTING.md is upfront about low acceptance rate for external contributions
- No dedicated DEVELOPMENT.md for advanced topics (debugging providers, session lifecycle)

### Build System: WELL-CONFIGURED

- Turbo orchestrates 4 core tasks: `build`, `dev`, `typecheck`, `test`
- Multiple dev modes: `dev` (full stack), `dev:server`, `dev:web`, `dev:desktop`
- Modern tooling: OxLint + OxFmt (Rust-based, fast), Vite 8 with React Compiler

**Assessment:** Excellent DX for contributors familiar with the stack. Missing a DEVELOPMENT.md for deeper topics and local browser test setup instructions.

---

## 6. Governance Readiness

### CONTRIBUTING.md: STRICT BUT CLEAR

- Not actively accepting large contributions
- Accepts: small bug fixes, reliability fixes, small perf improvements
- Rejects: 1000+ line PRs, drive-by features, opinionated rewrites
- Issue-first recommended for non-trivial changes

### CI Gates: COMPREHENSIVE

**CI pipeline (`ci.yml`):**
1. Format check (oxfmt)
2. Lint (oxlint)
3. Typecheck
4. Unit tests
5. Browser tests (Playwright/chromium)
6. Full desktop build verification
7. Release smoke test

**Release pipeline (`release.yml`):**
- Multi-platform builds (macOS arm64/x64, Linux, Windows)
- Apple notarization + Azure Trusted Signing
- npm CLI publish
- GitHub Release with artifacts

### PR Process: MATURE

- **PR template** with required sections (What Changed, Why, UI Changes, Checklist)
- **PR size labeling** — XS through XXL, auto-applied
- **Contributor trust system** (`pr-vouch.yml`) — trusted/unvouched/denounced labels
- **VOUCHED.td** — 27 trusted contributors listed
- **Issue templates** — detailed bug report and feature request forms

### Gaps

- **No CODEOWNERS file** — code ownership is implicit
- Could benefit from explicit "small and focused" examples in CONTRIBUTING.md

**Assessment:** Governance is mature for an early-stage project. CI gates enforce quality. The trust system and size labeling support the "small PRs only" policy. Adding CODEOWNERS would formalize module ownership.

---

## 7. Security & Operational Concerns

### Auth Tokens: WELL-HANDLED

- Desktop auth token: `Crypto.randomBytes(24).toString("hex")` — cryptographically secure
- Token validated on WebSocket upgrade (`wsServer.ts:935-948`)
- No hardcoded secrets found in codebase
- Sensitive values read from environment variables

### WebSocket Security: BASIC

- Token-based authentication on upgrade ✓
- Schema validation on all messages ✓
- **⚠ No Origin header validation** — potential Cross-Site WebSocket hijacking if token leaks
- **⚠ Token comparison uses direct string match** — should use `crypto.timingSafeEqual()`
- Mitigated by loopback-only binding in desktop mode

### Process Spawning: SAFE

- All process spawning uses array arguments (`spawn(command, args)`) — no shell injection
- Shell selected from predefined list (bash, zsh, cmd.exe, powershell.exe)
- Output size limits enforced (`DEFAULT_MAX_BUFFER_BYTES = 8MB`)
- Git commands use `ChildProcess.make("git", args)` — safe

### Sandboxing: NONE (BY DESIGN)

- Full terminal access with user's shell — expected for a dev tool
- File operations validated with path traversal prevention:
  - Path normalization + `.startsWith()` checks
  - `../` blocked, null bytes rejected
  - Attachment paths validated via `normalizeAttachmentRelativePath()`

### Input Validation: STRONG

- All WebSocket requests validated with Effect Schema
- Path traversal prevention on file operations
- Image attachments: MIME type, size limits, safe extensions, base64 validation
- `TrimmedNonEmptyString` schema for string inputs

### Recommendations

1. Add Origin header validation on WebSocket upgrade
2. Use `crypto.timingSafeEqual()` for token comparison
3. Add `npm audit` / `bun audit` to CI pipeline
4. Consider stricter git ref name validation for branch inputs

**Assessment:** Security posture is solid for a local development tool. No critical vulnerabilities found. The schema-first validation approach prevents most injection attacks. The WebSocket Origin and timing-safe comparison items are low-severity improvements.

---

## 8. Recommendations

### Summary of Key Findings

| Area | Rating | Key Issue |
|------|--------|-----------|
| Architecture | ✅ Strong | Clean event-sourcing, good boundaries |
| Provider Abstraction | ⚠ Moderate | Pluggable but `ProviderKind` type-locked to "codex" |
| Code Quality | ⚠ Moderate | 7 files >1200 lines, helper duplication |
| Testing | ⚠ Moderate | 30-35% coverage, no E2E, gaps in contracts |
| Developer Experience | ✅ Strong | Low friction, modern tooling |
| Governance | ✅ Strong | Mature CI, PR process, trust system |
| Security | ✅ Strong | No critical issues, schema-validated |

### Upstream Sync Workflows

Two Claude slash commands have been created in `.claude/commands/`:

1. **`sync-check`** — Checks if the fork is in sync with upstream, reports divergence
2. **`upstream-diff`** — Shows what changed upstream since a given commit (defaults to this review's HEAD)

### Recommendations for Ongoing Work

1. **Before starting the Ollama spike (Task 2):** Extend `ProviderKind` to a union type, and extract the canonical event mapping into a shared utility that adapters can reuse
2. **Helper deduplication:** Extract `asString`, `asObject`, `toTurnId`, `sameId` into `packages/shared`
3. **Test contracts:** Add tests for untested schema files (`project.ts`, `editor.ts`, `model.ts`, `baseSchemas.ts`)
4. **Large file refactoring:** `codexAppServerManager.ts` (1,587 lines) is the prime candidate for decomposition
5. **Add CODEOWNERS** for explicit module ownership
6. **Add `bun audit` to CI** for dependency vulnerability scanning
