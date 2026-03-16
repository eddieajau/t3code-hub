# Installation

One-off installation for a new machine (macOS only).
Once done see `README.md` for the daily dev workflow.

## Minimum system requirements

Apple Silicon M1+ with 16 GB RAM (24 GB+ recommended for local LLM testing).

## Prerequisites

### Bun

t3code uses [Bun](https://bun.sh) as its JavaScript runtime and package manager.

```bash
brew install oven-sh/bun/bun
bun --version   # verify installation
```

### Ollama (optional — for local LLM testing)

```bash
brew install ollama
```

Start/stop the Ollama service:

```bash
brew services start ollama   # start in background
brew services stop ollama    # stop when done
```

Pull a model:

```bash
ollama pull llama3.1      # 8B — ~4.7 GB, runs well on 16 GB RAM
ollama pull llama3.1:70b  # 70B — ~40 GB, needs 64 GB+ RAM
```

Visit <http://localhost:11434> to verify Ollama is running (you should see "Ollama is running").

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
