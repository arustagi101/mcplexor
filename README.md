# MCPlexor

**The Intelligent Multiplexer for MCP Servers.**

MCPlexor solves the "context window waste" problem by dynamically exposing tools to your AI agent only when they are needed. It acts as a smart proxy that sits between your agent (Claude, Cursor, Augment Code) and your MCP servers (Linear, Notion, Playwright, etc.).

> **☁️ MCPlexor Cloud** – Don't want to run your own inference? Join the [waitlist](https://mcplexor.com) for our hosted routing service. We run optimized small models so you get the savings without the infra.

## 🚀 The Problem: 40k Tokens of Bloat

If you're building agents with several MCP servers, you're dumping **~40k tokens** of tool definitions into your context on every request.

Here's what we measured:

| MCP Server | Tools | Tokens |
|------------|-------|--------|
| Notion | 22 | ~26k |
| Linear | 33 | ~8k |
| Playwright | 22 | ~5k |

That's **20% of a 200k context window** gone before your agent even processes the first message. And in practice, most runs only touch 1-2 tools – the rest is dead weight.

**The consequences:**
- 💸 **Higher API Costs**: Paying for tokens that do nothing
- 🐢 **Slower Responses**: More tokens = more latency
- � **Less Reasoning Room**: Context filled with unused tool definitions
- 🤯 **More Compaction**: Frequent context summarization triggered early

Anthropic wrote about this exact problem in their [Advanced Tool Use post](https://www.anthropic.com/engineering/advanced-tool-use). MCPlexor gives you the same experience for your own agents.

## 💡 The Solution: Semantic Discovery

MCPlexor acts as a single, lightweight MCP server that proxies to others. Instead of preloading every tool definition:

1. **Lightweight Index**: MCPlexor exposes a minimal set of capabilities based on your server descriptions.
2. **Semantic Routing**: When the agent needs a tool (e.g., "create a Linear issue"), MCPlexor uses an LLM to identify the correct server.
3. **Dynamic Loading**: Only the relevant server's tools get exposed to the agent.

**Result**: ~500 tokens overhead instead of ~40k. That's a **95%+ reduction**.

## 🎯 When to Use What

| Use Case | Recommendation |
|----------|----------------|
| **Power users / high volume** | Use **MCPlexor Cloud** ([waitlist](https://mcplexor.com)) – we run routing on optimized models, you get savings without running infra |
| **Trying it out / privacy-first** | Use **Ollama** backend – runs 100% locally, zero cost, works offline |
| **Claude Code / Augment Code** | MCPlexor exposes an "advanced search" tool that agents can call to discover capabilities dynamically |

## ✨ Key Features

- **Protocol Multiplexing**: Run multiple MCP servers (stdio or http) under one connection.
- **BYOK with Ollama**: Full support for local LLMs for the routing logic. Keep your data private.
- **Secure Credentials**: API keys stored in OS keychain (macOS Keychain / Windows Credential Manager / Linux keyring). Never in plaintext.
- **TUI & CLI**: Interactive Terminal UI for easy server management, or script with CLI.
- **Cross-Platform**: Works on macOS, Linux, and Windows.

## 📦 Installation

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
# Add Linear MCP Server
mcplexor mcp add linear -d "Linear for issue tracking and project management" -t http https://mcp.linear.app/mcp

# Add Notion MCP Server  
mcplexor mcp add notion -d "Notion for docs and knowledge base" -- npx -y @notionhq/mcp-server

# Add Playwright MCP Server
mcplexor mcp add playwright -d "Browser automation and testing" -- npx -y @anthropic/mcp-playwright
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

## 🧠 Local Setup with Ollama

MCPlexor supports "Bring Your Own Key" (BYOK), allowing you to use your own local LLM for routing. Zero cost, full privacy, works offline.

1. Ensure [Ollama](https://ollama.com/) is installed and running.
2. Pull a model (e.g., `llama3`, `qwen3`, or `mistral`).
3. In the TUI, press `/` then `model` and select `ollama`.

Or configure via `~/.mcplexor/config.json`:

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

## 🤖 Agent Rules (Recommended)

To get the best performance, add these instructions to your Agent's system prompt (e.g., `.cursorrules` or `.windsurfrules`):

```text
Always prefer to use the 'mcplexor' tool to discover available capabilities before attempting to use native tools or giving up.
If you are unsure how to proceed, use the search/find capability to discover relevant tools.
```