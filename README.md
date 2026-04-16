# claude-code-anywhere

Use [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) in VS Code (or any dev container) with an [OpenRouter](https://openrouter.ai) API key — no Anthropic subscription required. Includes MCP client support.

---

## Table of Contents

1. [How it works](#how-it-works)
2. [Quick start — Dev Container](#quick-start--dev-container)
3. [Manual setup — from scratch](#manual-setup--from-scratch)
   - [Step 1: Create an OpenRouter account](#step-1-create-an-openrouter-account)
   - [Step 2: Get your API key](#step-2-get-your-api-key)
   - [Step 3: Install Claude Code in VS Code](#step-3-install-claude-code-in-vs-code)
   - [Step 4: Configure the environment](#step-4-configure-the-environment)
   - [Step 5: Verify the connection](#step-5-verify-the-connection)
4. [MCP client support](#mcp-client-support)
5. [Using other models](#using-other-models)

---

## How it works

Claude Code normally talks to Anthropic's API. By setting a couple of environment variables, you can point it at OpenRouter instead. OpenRouter exposes an interface that is fully compatible with the Anthropic Messages API, so Claude Code doesn't know (or care) that it's talking to a different backend. You pay with OpenRouter credits rather than an Anthropic subscription.

```
VS Code / Claude Code  →  OpenRouter  →  Anthropic (or any other provider)
```

The three key variables:

| Variable | Value | Why |
|---|---|---|
| `ANTHROPIC_BASE_URL` | `https://openrouter.ai/api` | Redirects all API calls to OpenRouter |
| `ANTHROPIC_AUTH_TOKEN` | your OpenRouter API key | Authenticates with OpenRouter |
| `ANTHROPIC_API_KEY` | `""` (empty string) | Prevents Claude Code from complaining about a missing Anthropic key |

---

## Quick start — Dev Container

### Prerequisites

- [VS Code](https://code.visualstudio.com/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (running)
- [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) installed in VS Code
- An OpenRouter API key (see [Step 1](#step-1-create-an-openrouter-account) and [Step 2](#step-2-get-your-api-key) below if you don't have one yet)

### Setup

**1. Set the environment variables on your host machine.**

The dev container reads them from your host environment, so you never have to paste secrets inside the repo.

On **macOS / Linux**, add these lines to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export OPENROUTER_API_KEY="sk-or-xxxx-your-key-here"
export ANTHROPIC_MODEL="anthropic/claude-sonnet-4-6"   # or any model you prefer
```

Then reload the profile:

```bash
source ~/.zshrc   # or source ~/.bashrc
```

On **Windows** (PowerShell), set them as user environment variables:

```powershell
[System.Environment]::SetEnvironmentVariable("OPENROUTER_API_KEY","sk-or-xxxx-your-key-here","User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_MODEL","anthropic/claude-sonnet-4-6","User")
```

**2. Open this folder in VS Code, then reopen in container.**

When VS Code detects the `.devcontainer` folder it will show a notification:
> *"Folder contains a Dev Container configuration file. Reopen in Container?"*

Click **Reopen in Container**. Alternatively, open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`) and run:

```
Dev Containers: Reopen in Container
```

VS Code will build the image and install Claude Code automatically. This takes a minute on the first run; subsequent opens are instant.

**3. Open the Claude Code panel** from the Activity Bar and run `/status` to confirm the connection.

---

## Manual setup — from scratch

Follow these steps if you prefer not to use a dev container, or if you just want to understand the full setup.

### Step 1: Create an OpenRouter account

1. Go to [https://openrouter.ai](https://openrouter.ai) and click **Sign Up**.
2. Complete registration (email or OAuth with Google/GitHub).
3. Navigate to **Credits** in the top-right menu and add some credit. Even a few dollars goes a long way — Claude Sonnet costs roughly $0.003 per 1k input tokens. If you want to test first, OpenRouter has a small free allowance and also offers free-tier models (see [Using other models](#using-other-models)).

### Step 2: Get your API key

1. In the OpenRouter dashboard, click your avatar → **Keys**.
2. Click **Create Key**, give it a name (e.g. `vscode-claude`), and copy the key. It starts with `sk-or-`.
3. Store it somewhere safe — you won't be able to see it again.

### Step 3: Install Claude Code in VS Code

1. Open VS Code.
2. Go to the Extensions panel (`Ctrl+Shift+X` / `Cmd+Shift+X`).
3. Search for **Claude Code** (publisher: Anthropic) and click **Install**.
4. Reload VS Code if prompted.

Claude Code also requires **Node.js ≥ 18**. Check with:

```bash
node --version
```

If Node.js is not installed, download it from [https://nodejs.org](https://nodejs.org) (LTS version is fine).

### Step 4: Configure the environment

You have two options. **Option A** is persistent and recommended.

#### Option A — `~/.claude/settings.json` (persistent, applies everywhere)

Create or edit the file `~/.claude/settings.json`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api",
    "ANTHROPIC_AUTH_TOKEN": "sk-or-xxxx-your-key-here",
    "ANTHROPIC_API_KEY": "",
    "ANTHROPIC_MODEL": "anthropic/claude-sonnet-4-6"
  }
}
```

This file is read by Claude Code on every startup, on any project, without you having to remember to set anything.

#### Option B — Shell environment variables (per session)

Add the following to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export ANTHROPIC_AUTH_TOKEN="sk-or-xxxx-your-key-here"
export ANTHROPIC_API_KEY=""
export ANTHROPIC_MODEL="anthropic/claude-sonnet-4-6"
```

Then reload (`source ~/.zshrc`) and always **launch VS Code from the terminal**:

```bash
code .
```

This ensures VS Code inherits your shell environment. If you launch VS Code from the Dock or Start Menu it may not pick up the variables.

### Step 5: Verify the connection

Open any project folder in VS Code. Click the **Claude Code icon** in the Activity Bar to open the panel. If a sign-in screen appears, dismiss it — you don't need an Anthropic account when using OpenRouter.

In the Claude Code chat box, type:

```
/status
```

You should see your model and provider confirmed. If you get an authentication error, double-check that `ANTHROPIC_AUTH_TOKEN` contains your OpenRouter key and that `ANTHROPIC_API_KEY` is set to an empty string.

---

## MCP client support

Claude Code is an MCP **client** out of the box. This means it can connect to any [Model Context Protocol](https://modelcontextprotocol.io) server — giving it tools that extend beyond coding: reading files, querying databases, calling APIs, browsing the web, and much more.

This setup works with MCP with no extra configuration. The OpenRouter routing is transparent to the MCP layer.

### How MCP fits in

```
Claude Code (MCP client)
    ├── OpenRouter  →  LLM (Claude, Llama, etc.)
    └── MCP servers →  tools (filesystem, GitHub, Postgres, browser…)
```

Claude Code uses the LLM to decide *when* to call a tool, and MCP servers to actually *execute* it. The two channels are independent — swapping the LLM backend via OpenRouter has no effect on MCP connectivity.

### Adding an MCP server

MCP servers are registered in `~/.claude/mcp.json`. Create the file if it doesn't exist:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/your/project/path"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    }
  }
}
```

Or register a server from the CLI:

```bash
claude mcp add filesystem npx -- -y @modelcontextprotocol/server-filesystem /your/project
```

Run `/mcp` inside Claude Code to see all connected servers and their status.

### Adding MCP servers to the dev container

To pre-install MCP servers inside the container, extend `postCreateCommand` in `devcontainer.json`:

```json
"postCreateCommand": "npm install -g @anthropic-ai/claude-code @modelcontextprotocol/server-filesystem @modelcontextprotocol/server-github"
```

Then place your `mcp.json` in the repo root and mount it into the container by adding to `devcontainer.json`:

```json
"mounts": [
  "source=${localWorkspaceFolder}/mcp.json,target=/root/.claude/mcp.json,type=bind,consistency=cached"
]
```

This way the entire Claude Code environment — LLM backend, model choice, and MCP tools — is fully defined in the repo and reproducible on any machine.

### Useful MCP servers to get started

| Server | npm package | What it gives Claude |
|---|---|---|
| Filesystem | `@modelcontextprotocol/server-filesystem` | Read/write files outside the workspace |
| GitHub | `@modelcontextprotocol/server-github` | Issues, PRs, commits, repo search |
| PostgreSQL | `@modelcontextprotocol/server-postgres` | Query and inspect a Postgres database |
| Brave Search | `@modelcontextprotocol/server-brave-search` | Live web search |
| Memory | `@modelcontextprotocol/server-memory` | Persistent key-value memory across sessions |

A larger catalog is available at [https://github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers).

---

## Using other models

OpenRouter gives you access to hundreds of models from different providers. To switch model, change the `ANTHROPIC_MODEL` variable (in `settings.json` or your shell profile) to any model ID from the [OpenRouter model catalog](https://openrouter.ai/models).

### Recommended Claude models (via OpenRouter)

| Model | ID | Best for |
|---|---|---|
| Claude Sonnet 4.6 | `anthropic/claude-sonnet-4-6` | General coding, default choice |
| Claude Opus 4.6 | `anthropic/claude-opus-4-6` | Complex reasoning, architecture work |
| Claude Haiku 4.5 | `anthropic/claude-haiku-4-5` | Fast completions, high volume |

### Free-tier models (no credit needed)

OpenRouter offers several capable models at $0 per token, with rate limits. Filter by **Free** on the [model catalog](https://openrouter.ai/models).

| Model | ID |
|---|---|
| Meta Llama 3.3 70B | `meta-llama/llama-3.3-70b-instruct:free` |
| DeepSeek R1 | `deepseek/deepseek-r1:free` |
| Google Gemma 3 12B | `google/gemma-3-12b-it:free` |

> **Note:** Free models are capable but are not fine-tuned for agentic coding tasks the way Claude models are. Expect occasional tool-use failures or instruction-following issues, especially on multi-file edits. They work well for explanation, review, and simple completions.

### Non-Claude models (paid)

OpenRouter also has GPT, Gemini, Mistral, and others. You can point Claude Code at any of them using the same setup — just replace the model ID.

To switch between models on the fly without editing config files, you can define shell aliases:

```bash
alias cc-sonnet='ANTHROPIC_MODEL="anthropic/claude-sonnet-4-6" claude'
alias cc-opus='ANTHROPIC_MODEL="anthropic/claude-opus-4-6" claude'
alias cc-free='ANTHROPIC_MODEL="meta-llama/llama-3.3-70b-instruct:free" claude'
```

Add these to your shell profile and you can pick a model just by choosing which alias to run.

---

> **Cost tip:** Check the [OpenRouter Activity Dashboard](https://openrouter.ai/activity) to monitor your spending in real time. Set a monthly credit limit under **Settings → Limits** to avoid surprises.
