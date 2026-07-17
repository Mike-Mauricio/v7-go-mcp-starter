# V7 Go MCP Starter

Build V7 Go agents and workflows from your AI coding tool — no code required.

This starter kit connects your AI assistant (Claude Code, Cursor, or Codex) to your V7 Go workspace through the [V7 MCP](https://docs.go.v7labs.com/docs/connect-go-to-claude-chatgpt-and-other-ai-assistants). Once connected, you can design, build, run, and query V7 Go workflows through natural conversation.

## What You Can Do

- **Build workflows** — Tell your AI assistant what documents you work with and what you need from them. It designs the workflow and creates it in V7 Go.
- **Query data** — Ask about your existing workflows, check processing results, and explore your workspace.
- **Run workflows** — Trigger workflow runs directly from your AI tool, including with file uploads.
- **Manage Knowledge Hubs** — Upload documents and query your Knowledge Hubs in natural language.

## Quick Start (5 Minutes)

### 1. Clone this repo

```bash
git clone https://github.com/Mike-Mauricio/v7-go-mcp-starter.git
cd v7-go-mcp-starter
```

### 2. Connect to V7 Go

Copy the MCP configuration template:

```bash
cp .mcp.json.example .mcp.json
```

The `.mcp.json` file connects your AI tool to V7 Go's MCP server:

```json
{
  "mcpServers": {
    "v7-go": {
      "type": "http",
      "url": "https://mcp.go.v7labs.com"
    }
  }
}
```

> **US region?** Change the URL to `https://mcp.go.us.v7labs.com`

### 3. Open in your AI tool

**Claude Code:**
```bash
claude
```

**Cursor:** Open the folder in Cursor — it reads `.cursorrules` automatically.

**Codex:** Open the folder — it reads `AGENTS.md` automatically.

### 4. Authenticate

When your AI tool starts, it will connect to the V7 MCP server and prompt you to authenticate via your browser. Log in with your V7 Go credentials and select your workspace.

### 5. Build your first workflow

Tell your AI assistant:

> "I want to build a workflow that extracts company names and dates from PDF contracts and puts them in a table."

It will walk you through the design, confirm your choices, and create the workflow in your V7 Go workspace.

## What's in This Repo

| Folder | What's Inside |
|--------|--------------|
| `CLAUDE.md` | AI instructions for Claude Code (the AI reads this to understand V7 Go) |
| `.cursorrules` | Same instructions adapted for Cursor |
| `AGENTS.md` | Same instructions adapted for Codex |
| `CONTEXT-INDEX.md` | Task-based lookup — tells the AI which files to read for which task |
| `skills/` | Guided workflows: build-workflow, design-agent, explore-data, getting-started |
| `references/` | Platform guide, prompt patterns, model selection, workflow patterns, views, triggers, skills architecture, gotchas, integrations, regional setup, API endpoints, example agents |
| `references/advanced/` | Advanced references: rebuild playbooks, SharePoint integration, cross-workspace operations |
| `templates/` | Workflow spec, use case brief, workspace resources tracker |
| `docs/` | Quickstart guide, best practices, FAQ |

## Reference Guides

**Getting Started:**
- [Quickstart](docs/quickstart.md) — 5-minute first workflow guide
- [Platform Guide](references/v7-go-platform-guide.md) — What V7 Go is and how it works
- [FAQ](docs/faq.md) — Common questions and troubleshooting

**Building Workflows:**
- [Prompt Patterns](references/prompt-patterns.md) — 7 reusable prompt templates for agent properties
- [Model Selection](references/model-selection-guide.md) — Which AI model for which task
- [Property Types](references/property-types-reference.md) — All property types, tools, and configurations
- [Workflow Patterns](references/workflow-patterns.md) — 8 production patterns including LLM-as-a-Judge
- [Views](references/views-reference.md) — Routing, triage, and conditional processing
- [Best Practices](docs/best-practices.md) — 10 design principles from production workflows
- [Gotchas](references/gotchas.md) — 74 platform gotchas to avoid common mistakes

**Connecting & Automating:**
- [Integration Guide](references/integration-guide.md) — MCP Connectors, Zapier, API, Concierge
- [Triggers](references/triggers-reference.md) — Email, schedule, and webhook triggers
- [Skills Architecture](references/skills-architecture.md) — Building V7 Go platform skills
- [Regional Setup](references/regional-setup.md) — EU vs US regions and data residency
- [API Endpoints](references/api-endpoints.md) — Complete API endpoint reference

**Advanced (for solutions engineers):**
- [Rebuild Playbooks](references/advanced/rebuild-playbooks.md) — Agent migration and rebuild procedures
- [SharePoint Integration](references/advanced/sharepoint-integration.md) — Enterprise SharePoint architectures
- [Cross-Workspace](references/advanced/cross-workspace.md) — Multi-workspace operations and isolation

## Requirements

- A [V7 Go](https://go.v7labs.com) workspace with an active account
- One of: [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Cursor](https://cursor.sh), or [Codex](https://openai.com/codex)
- That's it. No coding experience needed.

## About V7 Go

[V7 Go](https://www.v7labs.com) is an AI agent platform for automating complex, document-intensive workflows. It's used by enterprise teams in financial services, real estate, insurance, and legal to process documents with 95-99% accuracy — with full auditability through AI Citations that trace every extracted value back to its source.

## License

MIT
