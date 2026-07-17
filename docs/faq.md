# Frequently Asked Questions

## Getting Started

### What is V7 Go?
V7 Go is an AI agent platform for automating complex, document-intensive workflows. Think of it as a team of AI specialists that can read documents, extract data, generate reports, and connect to your existing business tools — all without writing code.

### What is the V7 MCP?
MCP (Model Context Protocol) is a standard that lets AI tools communicate with external platforms. The V7 MCP server connects your AI coding tool (Claude Code, Cursor, Codex) directly to your V7 Go workspace. This means you can build, run, and query V7 Go workflows by having a conversation with your AI assistant instead of clicking through the V7 Go interface.

### Do I need to know how to code?
No. V7 Go is a no-code platform, and this starter kit is designed for non-technical users. Your AI assistant handles the technical details — you focus on describing what you need.

### Which AI tool should I use?
This starter kit works with:
- **Claude Code** — Full skill support + guided workflows (recommended for the best experience)
- **Cursor** — Uses .cursorrules for context
- **Codex** — Uses AGENTS.md for context

All three can connect to V7 Go via the MCP and help you build workflows.

---

## Building Workflows

### What's the difference between a "workflow" and an "agent" in V7 Go?
They're the same thing. A V7 Go workflow (also called an agent or project) is a multi-step AI pipeline that processes documents or data. Each step is called a "property" — it takes an input, does something with it (extract, classify, summarize, etc.), and produces an output.

### Which model should I use?
It depends on the task. Here's the quick guide:

| Task | Recommended Model | Why |
|------|------------------|-----|
| Simple extraction (names, dates, numbers) | `gemini_3_flash` | Fast, cheap, accurate |
| Classification and tagging | `gpt_5_mini` or `claude_4_5_haiku` | Good at structured categories |
| Structured JSON output | `gpt_5_5` | Reliable JSON formatting |
| Complex analysis and reasoning | `claude_4_6_sonnet` | Strong at nuanced judgment |
| Knowledge Hub search | `gpt_5_5` | Best at hub query synthesis |
| Long document summarization | `gemini_3_1_pro` | Large context window |
| Compliance and legal review | `claude_4_6_sonnet` | Strong at rule-based evaluation |
| Deep reasoning / complex code | `claude_4_7_opus` | Most capable for hard tasks |

See [references/model-selection-guide.md](../references/model-selection-guide.md) for the full matrix.

### What is "grounding" and when should I use it?
Grounding means the AI shows exactly where in the source document it found each piece of data — with visual bounding boxes on the original PDF. This is called an **AI Citation**.

**Use grounding when:**
- Extracting specific data from documents (names, numbers, dates, clauses)
- Compliance or audit requirements demand traceability
- Users need to verify AI outputs against the source

**Don't use grounding when:**
- Generating new text (summaries, analysis, recommendations)
- Classifying or categorizing (no specific source location to point to)
- Using web search or Knowledge Hub inputs (not a document extraction task)

### How do I handle multi-page documents?
V7 Go's Index Knowledge technology automatically breaks large documents into searchable chunks. You don't need to split documents manually. Just upload the full document and write your prompts as if the AI can see the whole thing — it can.

For very large documents (100+ pages), best practices:
- Be specific about where to look: "In the Fee Schedule section, extract..."
- Use Knowledge Hubs for reference documents the AI should consult across multiple workflows

### What is "thinking effort" and how do I set it?
Thinking effort controls how much reasoning the AI model applies to each task. Higher effort = more accurate but slower and more expensive.

| Level | When to Use |
|-------|------------|
| `minimal` | Simple lookups, tagging, yes/no decisions |
| `low` | Standard extraction, straightforward classification |
| `medium` | Multi-step reasoning, comparisons, analysis |
| `high` | Complex legal/compliance review, deep analysis |

**Important:** Not all models support all levels. See [references/model-selection-guide.md](../references/model-selection-guide.md) for the compatibility matrix.

### How should I write prompts for V7 Go properties?
Follow these patterns:

1. **Be specific about what to extract**: "Extract the management fee percentage" not "Get the fees"
2. **Always include N/A handling**: "If not found, respond with N/A"
3. **Specify the output format**: "Format as YYYY-MM-DD" or "Format as a percentage (e.g., 1.5%)"
4. **Reference inputs explicitly**: Use `@PropertyName` to tell the AI which input to look at

See [references/prompt-patterns.md](../references/prompt-patterns.md) for 7 complete prompt templates.

---

## Outputs and Integrations

### Can V7 Go create documents (Word, PDF, Excel)?
Yes. V7 Go can generate structured documents as workflow outputs:
- **Excel/CSV** — Structured data tables that can be downloaded or exported
- **Word/PDF** — Generated documents using AI Docs
- **Presentations** — Via integration with external tools

### How do I connect V7 Go to my other tools?
Three ways:

1. **MCP Connectors** (built into V7 Go) — Direct connections to HubSpot, Salesforce, Slack, Google Drive, and 400+ other apps. Toggle on in Settings → Integrations.

2. **Zapier** (8,000+ apps) — Set up triggers and actions between V7 Go and virtually any business tool. No code required.

3. **API** (for custom integrations) — V7 Go has a REST API for programmatic access. Useful if your team has developers who want to build custom pipelines.

### Can I connect to my CRM?
Yes. V7 Go connects to Salesforce, HubSpot, and other CRMs via MCP connectors or Zapier. Common patterns:
- Push extracted deal terms into CRM records
- Update contact information from processed documents
- Create new records when documents match certain criteria

---

## Troubleshooting

### My MCP connection isn't working
1. Check that V7 Go is added as a connector in your AI tool's settings (Settings → Connectors → Add Custom Connector)
2. Verify you're using the correct regional URL:
   - EU: `https://mcp.go.v7labs.com`
   - US: `https://mcp.go.us.v7labs.com`
3. Try disconnecting and reconnecting the V7 Go connector
4. Complete the OAuth authentication in your browser when prompted
5. Check that you selected the correct workspace during auth

### The AI extracted the wrong data
Common causes and fixes:
- **Prompt too vague** → Be more specific about what to extract and where to look
- **Wrong model** → Try a more capable model (e.g., Claude 4.6 Sonnet for complex documents)
- **Thinking effort too low** → Increase from `low` to `medium`
- **No N/A handling** → Add "If not found, respond with N/A" to prevent hallucination
- **Grounding not enabled** → Turn on grounding for document extraction properties

### The workflow is slow
- Lower thinking effort for simple properties (extraction doesn't need `high`)
- Use faster models for simple tasks (Gemini 3 Flash, GPT-5 Mini)
- Split complex properties into smaller, focused ones
- Check if properties are referencing more inputs than they need
