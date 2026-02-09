# MCPlexor

**The Intelligent Multiplexer for MCP Servers.**

MCPlexor solves the "context window waste" problem by dynamically exposing tools to your AI agent only when they are needed. It acts as a smart proxy that sits between your agent (Claude, Cursor, Windsurf) and your heavy MCP servers (Linear, GitHub, Postgres, etc.).

## 🚀 The Problem: Token Waste

Modern AI agents are powerful, but they have a critical limitation: **Context Window**.

- Heavy MCP servers like **Linear** or **PostHog** can consume **10,000+ tokens** individually just to list their tool definitions.
- Connecting multiple such servers (e.g., GitHub + Linear + Postgres + Slack) can easily fill up 30,000-50,000 tokens of context before you even ask a question.
- This leads to:
  - 💸 **Higher API Costs**: Paying for thousands of unnecessary tokens on every request.
  - 🐢 **Slower Responses**: Agents process massive prompts unnecessarily.
  - 😵 **Confused Agents**: Too many tools overwhelm the agent, causing hallucinations or incorrect tool usage.

### 📉 Scenario: The 20k Token Tax

Imagine you want your agent to "Check my Linear issues and update the relevant code in GitHub."

| **Without MCPlexor** | **With MCPlexor** |
| :--- | :--- |
| Agent loads **ALL** Linear tools (~10k tokens) | Agent loads **ONLY** MCPlexor index (~100 tokens) |
| Agent loads **ALL** GitHub tools (~10k tokens) | MCPlexor dynamically routes to Linear/GitHub |
| **Total Overhead: ~20,000 tokens** | **Total Overhead: ~200 tokens** |
| 💸 **Cost**: High | 💰 **Cost**: Negligible |

## 💡 The Solution: Semantic Discovery

MCPlexor acts as a single, lightweight MCP server that proxies to others. Instead of dumping 500+ tool definitions into your agent's context:

1. **Lightweight Index**: MCPlexor exposes a minimal set of capabilities based on your server descriptions.
2. **Semantic Routing**: When the agent needs a tool (e.g., "check recent issues"), MCPlexor uses a lightweight routing model to identify the correct server.
3. **Dynamic Loading**: It then exposes only the relevant tools for that specific task.

**Result**: Massive token savings (often 90% reduction in system prompt size) and focused agent performance.

## ✨ Key Features

- **Protocol Multiplexing**: Run multiple MCP servers (stdio or http) under one connection.
- **BYOK with Ollama**: Full support for local LLMs (Ollama) for the routing logic. Keep your data private and avoid external API calls.
- **TUI & CLI**: Interactive Terminal UI for easy server management.
- **Cross-Platform**: Works on macOS, Linux, and Windows.

## 📦 Installation

To install the latest version:

```bash
curl -fsSL https://mcplexor.com/install.sh | bash
```

Or download manually from [GitHub Releases](https://github.com/arustagi101/mcplexor/releases).

## 🛠️ Usage

### 1. Add Servers

You can manage servers interactively with the TUI:

```bash
mcplexor
```

Or use the CLI to add servers directly:

```bash
# Add GitHub MCP Server
mcplexor mcp add github -d "GitHub API for managing repositories, issues, pull requests, and code search" -- npx -y @modelcontextprotocol/server-github

# Add Linear MCP Server
mcplexor mcp add linear -d "Linear for issue tracking and project management" -- npx -y @modelcontextprotocol/server-linear

# Add Postgres MCP Server
mcplexor mcp add postgres -d "PostgreSQL database for SQL queries" -- npx -y @modelcontextprotocol/server-postgres postgresql://localhost/mydb
```

### 2. Start MCPlexor

Launch the server to start proxying:

```bash
mcplexor serve
```

### 3. Connect Your Client

Configure your AI agent to use MCPlexor as its primary MCP server.

**For Claude Desktop:**
Edit `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "mcplexor": {
      "command": "mcplexor",
      "args": ["serve"]
    }
  }
}
```

**For Augment Code / Claude Code / Cursor:**
Go to **Settings > General > MCP Servers** and add a new server:
- **Name**: `mcplexor`
- **Type**: `command` (stdio)
- **Command**: `mcplexor serve`

## 🧠 Configuration: BYOK with Ollama

MCPlexor supports "Bring Your Own Key" (BYOK) paradigm, allowing you to use your own local LLM for the intelligent routing logic. This means zero cost for discovery and full privacy.

1. Ensure [Ollama](https://ollama.com/) is installed and running.
2. Pull a suitable model (e.g., `llama3` or `mistral`).
3. Configure MCPlexor to use Ollama by editing `~/.mcplexor/config.json`:

```json
{
  "settings": {
    "search": {
      "backend": "ollama",
      "ollama": {
        "url": "http://localhost:11434",
        "model": "llama3"
      }
    }
  },
  "mcpServers": {
    ...
  }
}
```

Now MCPlexor will use your local Ollama instance to "think" about which tool to serve, keeping your data strictly local.

## 🤖 Agent Rules (Recommended)

To get the best performance, add these instructions to your Agent's system prompt or custom instructions (e.g., `.cursorrules` or `.windsurfrules`):

```text
Always prefer to use the 'mcplexor' tool to discover available capabilities before attempting to use native tools or giving up.
If you are unsure how to proceed, use the search/find capability to discover relevant tools.
If the agent is stuck or hallucinating tools, explicitly type "use mcplexor" at the end of your message to force a fresh tool search.
```