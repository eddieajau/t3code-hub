# Installation

One-off installation for a new machine (macOS only).
Once done see `README.md` for the daily dev workflow.

## Prerequisites

### Bun

t3code uses [Bun](https://bun.sh) as its JavaScript runtime and package manager.

```bash
brew install oven-sh/bun/bun
```

### Ollama (optional — for local LLM testing)

```bash
brew install ollama
ollama serve          # start the server (runs on localhost:11434)
ollama pull llama3.1  # download a model (~4.7 GB)
```

## Clone and install

Clone the t3code repo as a sibling directory (the VS Code workspace file assumes `../t3code`):

```bash
git clone https://github.com/pingdotgg/t3code.git ../t3code
```

Install dependencies:

```bash
cd ../t3code && bun install
```

Verify the build works:

```bash
bun run typecheck
```
