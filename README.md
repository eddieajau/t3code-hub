# T3 Code: Hub

A workspace for exploring agentic guardrails that can be put around T3Code, governance experiments, and the stuff that doesn't fit upstream.
Consider this the bazaar that ships before the cathedral is ready.

Source: [t3code](https://github.com/pingdotgg/t3code)

## Goal

T3Code is a promising open-source coding agent — but today it only talks to cloud APIs. We want to make it work with local LLMs (starting with Ollama) so that developers can run a fully local-first, zero-cost coding assistant with no API keys and no data leaving their machine.

Starting with Ollama isn't about competing with cloud models on intelligence — it's about proving the interaction layer works. Once a local model can manage a session, stream responses, and participate in the provider lifecycle, the door opens wider: any contributor with a laptop can hack on the agent without API costs, and smaller local models become useful for the "dumb but frequent" jobs that don't need frontier intelligence — tool-call routing, indexing conversation history, generating commit summaries, and other harness-level tasks where latency and cost matter more than raw capability.

The changes needed to support this touch contracts, server wiring, a new provider adapter, and UI — too many moving parts to land incrementally through upstream PRs without a clear adapter pattern to target. Rather than death-by-a-thousand-patches, we're proving the concept end-to-end in a true fork first. If it works, the findings and (hopefully) a clean adapter interface can inform a contribution back upstream.

## Disclaimer

**This is an independent, unofficial community project.** It is not affiliated with, endorsed by, or maintained by [ping.gg](https://ping.gg) or the t3code maintainers. The upstream project lives at [pingdotgg/t3code](https://github.com/pingdotgg/t3code). All original t3code code remains under its own license.
