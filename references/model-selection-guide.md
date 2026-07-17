# V7 Go Model Selection & Tool Configuration Guide

## Core Principle

**Use the best model for the task, not a default provider.** Vary models across properties — a single agent should use GPT, Gemini, and Claude where each excels. The examples may lean GPT-heavy because they were built iteratively; new agents should be smarter about model selection from the start.

## Task-to-Model Decision Matrix

| Task Type | Primary Pick | Alternate | Thinking Effort | Grounded | Why |
|---|---|---|---|---|---|
| **Classification / tagging** | `gemini_3_flash` | `gpt_5_mini` or `claude_4_5_haiku` | `minimal` | No | Fastest, cheapest — all three work well |
| **Simple extraction** (single field) | `gemini_3_flash` | `gpt_5_5` | `low` | Yes | Gemini Flash is fast and cheap for straightforward pulls |
| **Complex extraction** (multi-field, judgment) | `gpt_5_5` | `claude_4_6_sonnet` | `medium` | Yes | Both strong at structured reasoning |
| **Structured JSON output** | `gpt_5_5` | `gpt_5_mini` | `low` to `medium` | Yes | GPT excels at reliable JSON formatting |
| **Document generation / synthesis** | `claude_4_6_sonnet` | `gemini_3_1_pro` | `medium` | No | Claude for structure + polish, Gemini for narrative flow |
| **Long-form narrative writing** | `gemini_3_1_pro` | `gemini_3_flash` | `medium` | No | Gemini best at natural, flowing prose |
| **Compliance review / redlines** | `claude_4_6_sonnet` | `gpt_5_5` | `medium` | Yes | Claude excels at careful, structured analysis |
| **Code / HTML generation** | `claude_4_6_sonnet` | `claude_4_7_opus` | `medium` | No | Anthropic strongest for code by a wide margin |
| **Data parsing / formatting** | `code` | — | N/A | No | Zero token cost — always prefer Python |
| **Web research / enrichment** | `web_search` | — | `low` | No | External data lookup |
| **Hub-based analysis** | `gpt_5_5` | `gpt_5_mini` | `low` to `medium` | No | GPT best at tool calling / hub search |
| **Summary / rollup** | `gemini_3_flash` | `gpt_5_5` | `low` | No | Gemini produces clean, concise summaries |
| **Final classification / scoring** | `gpt_5_mini` | `gemini_3_flash` or `claude_4_5_haiku` | `minimal` | No | Lightweight, deterministic — cheapest option |

## Model Provider Strengths

| Provider | Best At | Use For | Avoid For |
|---|---|---|---|
| **GPT-family** | Structured JSON, tool calling, hub search, classification | JSON extraction, hub queries, structured outputs | Long narrative writing (tends to be formulaic) |
| **Gemini-family** | Narrative writing, summarization, long-context analysis, fast extraction, image generation | Summaries, narratives, simple extraction, classification, image output (`gemini_3_1_flash_image`) | Complex multi-tool workflows |
| **Anthropic (Claude)** | Code generation, complex reasoning, compliance review, document synthesis | Code/HTML, redline analysis, structured document generation, nuanced judgment, fast classification (`claude_4_5_haiku`) | Simple classification with Opus/Sonnet (overpriced for the task — use Haiku instead) |

## Model Tiers

| Tier | OpenAI | Google | Anthropic | Best For |
|---|---|---|---|---|
| **Light / Fast** | `gpt_5_mini` | `gemini_3_flash` | `claude_4_5_haiku` | Classification, tagging, simple extraction, summaries |
| **Mid-Tier** | `gpt_5_5` | `gemini_3_1_pro` | `claude_4_6_sonnet` | Most extraction, analysis, and document generation |
| **Strong** | `gpt_5_5` (high) | `gemini_3_1_pro` (high) | `claude_4_6_sonnet` (high) | Complex multi-step judgment, compliance review |
| **Top** | `gpt_5_5` (high) | — | `claude_4_7_opus` | Deep reasoning, math, complex code |

## Mixed-Model Pipeline Example

A well-designed agent should use multiple providers. Example for a document extraction agent:

| Property | Model | Why |
|---|---|---|
| Document Classification | `gemini_3_flash` (minimal) | Fast, cheap routing |
| Key Data Extraction (JSON) | `gpt_5_5` (low) | Reliable structured JSON |
| Compliance Check | `claude_4_6_sonnet` (medium) | Careful, nuanced analysis |
| Summary | `gemini_3_flash` (low) | Clean, concise narrative |
| Overall Score | `gpt_5_mini` (minimal) | Simple classification |

## Thinking Effort Levels

**IMPORTANT: Supported levels vary by model. Using the wrong level returns an API error.**

| Model | Lowest | Supported Levels |
|---|---|---|
| `gpt_5_5` | Check V7 Go docs for current compatibility | Check V7 Go docs for current compatibility |
| `gpt_5_mini` | `minimal` | minimal, low, medium, high |
| `gemini_3_1_pro` | Check V7 Go docs for current compatibility | Check V7 Go docs for current compatibility |
| `gemini_3_flash` | `minimal` | minimal, low, medium, high |
| `claude_4_7_opus` | `disabled` | disabled, medium, high (**no `low`!**) |
| `claude_4_6_sonnet` | `disabled` | disabled, medium, high (**no `low`!**) |
| `claude_4_5_haiku` | Check V7 Go docs for current compatibility | Check V7 Go docs for current compatibility |

**Safe defaults**: `low` works on most models. Use `minimal` only with GPT-5 Mini/Gemini 3 Flash. Use `disabled` only with Claude. Never use `low` with Claude — jump from `disabled` to `medium`.

**Default recommendation**: Start with `low` for most models. For Claude, start with `disabled` and escalate to `medium` or `high` when results are insufficient.

## When to Use `is_grounded: true`

Enable AI Citations (`is_grounded: true`) when:
- Extracting data directly from source documents (PDFs, files)
- Accuracy verification is important (financial data, legal terms, compliance)
- Users need to trace outputs back to source locations

Do NOT enable when:
- Generating new content (summaries, narratives, analysis)
- Classifying or routing (single_select outputs)
- Aggregating from multiple prior properties (no single source)
- Using web_search or code tools

## Cost Optimization Strategies

1. **Bundle fields using JSON type** — Extract multiple values in one LLM call instead of separate properties
2. **Use `code` tool for parsing** — Zero token cost for data transformation, formatting, validation
3. **Classify with single_select** — Cheaper and more reliable than open-ended text for routing
4. **Reuse prior property outputs** — Reference earlier properties instead of re-querying the document
5. **Use light models for light tasks** — `gemini_3_flash`, `gpt_5_mini`, or `claude_4_5_haiku` for classification, not mid-tier models
6. **Lower thinking effort first** — Start with `minimal` or `low`, escalate only if results are weak
