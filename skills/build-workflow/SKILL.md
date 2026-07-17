---
name: build-workflow
description: >-
  Guided workflow builder for V7 Go. Use when the user wants to create a new
  workflow, agent, or automation. Walks through identifying inputs, designing
  steps, selecting models, and deploying via the V7 MCP.
argument-hint: "[description of what you want to automate]"
---

# Build Workflow

You are guiding a non-technical user through building a V7 Go workflow (agent). Walk through each step below in order. Be conversational — one question at a time, plain language, no jargon.

Before starting, verify the V7 MCP connection is active. If it is not connected, stop and guide the user through setup (see skills/getting-started/).

## Step 1: Understand the Goal

Ask these questions **one at a time** — wait for each answer before asking the next:

1. "What documents or data do you work with?" (PDFs, scans, spreadsheets, emails, etc.)
2. "What do you currently do with them manually?" (extract data, review for compliance, generate a report, classify and route, etc.)
3. "What should the output look like?" (a data table, a formatted document, a classification/routing decision)
4. "Where does the output need to go?" (stay in V7 Go, push to Slack/email/CRM, download as Excel/PDF)

Summarize what you heard back to the user in 2-3 sentences before moving on.

## Step 2: Choose the Output Pattern

Based on the user's answers, identify which pattern fits:

| Pattern | Description | Example |
|---------|-------------|---------|
| **Structured Data** | Documents in, organized data table out | Extract financial data from PDFs into rows and columns |
| **Document Generation** | Documents in, new document out | Turn raw reports into formatted memos or Excel files |
| **Human-AI Augmentation** | AI handles 80-90%, humans review the rest | Auto-approve simple claims, route complex ones to a reviewer |

Tell the user which pattern you recommend and why. Get their confirmation before proceeding.

## Step 3: Design the Pipeline

Map out the properties (steps) the workflow needs, following this sequence:

```
Input -> Classify -> Extract -> Analyze -> Synthesize -> Status
```

Not every workflow needs all six stages. A simple extraction might only need Input, Extract, and Status.

For each property, determine:
- **Name** — Clear, descriptive (e.g., "Document Type", "Lease Start Date", "Risk Assessment")
- **Type** — file, text, single_select, multi_select, json, collection, number, url, hub_select
- **Tool** — Which AI model or tool powers it (see Step 4)
- **Grounded** — Whether AI Citations should be enabled (see best practices below)
- **Prompt** — What instructions the model needs (see Step 5)

**Best practices to apply automatically:**

- **One property, one task.** If a prompt needs to do 3+ things, split it into separate properties.
- **N/A handling in every extraction prompt.** Always include: "If not found, respond with N/A."
- **Selective grounding.** Enable AI Citations (`is_grounded: true`) for extraction from documents. Disable for summaries, classifications, and generated content.
- **Start thinking effort at low.** The `low` setting works on all models. Only increase to `medium` or `high` when results are not accurate enough.
- **Normalize output formats.** Specify date formats (YYYY-MM-DD), percentages (1.5%), currency ($150M) in every extraction prompt.
- **Use Python for zero-cost logic.** Formatting, parsing, calculations, and conditional routing belong in `code` properties, not LLM calls.

## Step 4: Select Models

Read `references/model-selection-guide.md` for the full decision matrix. Quick reference:

| Task | Model | Thinking Effort |
|------|-------|----------------|
| Simple extraction | `gemini_2_5_flash` or `gemini_3_flash` | `low` |
| Classification / tagging | `gemini_3_flash` or `gpt_5_mini` | `minimal` |
| Structured JSON output | `gpt_5` | `low` |
| Complex multi-field extraction | `gpt_5_1` or `claude_4_5_sonnet` | `medium` |
| Compliance review / redlines | `claude_4_5_sonnet` | `medium` |
| Summary / narrative | `gemini_3_flash` or `gemini_3_pro` | `low` |
| Knowledge Hub search | `gpt_5` | `low` to `medium` |
| Code / HTML generation | `claude_4_5_sonnet` | `medium` |
| Data parsing / formatting | `code` (Python) | N/A (free) |

**Key rule:** A well-designed workflow uses multiple models. Use the best model for each task, not a single default. A typical workflow might use Gemini for classification, GPT for extraction, Claude for compliance, and Python for formatting.

**Thinking effort compatibility:**
- `minimal` only works with GPT-5/GPT-5 Mini/Gemini 3 family
- `disabled` works with GPT-5.1/Gemini 2.5/Claude
- `low` works on all models (safe default)
- Never use `low` with Claude — jump from `disabled` to `medium`

## Step 5: Write Prompts

Read `references/prompt-patterns.md` for the full library of prompt templates. Apply these rules to every prompt:

1. **Assign a specific expert role** — "You are an expert insurance underwriter" not "You are a helpful assistant"
2. **Include N/A handling** — Tell the model what to output when data is missing
3. **Specify the output format** — Markdown, JSON, plain text, bullet list, etc.
4. **Reference inputs explicitly** — Use `@<Pproperty_ID>` to point to earlier properties or uploaded files
5. **Add exclusions** — What the model should NOT do or include
6. **Include formatting rules** — Date formats, number formats, currency, units

For each property, draft the prompt and show it to the user. Explain what it does in plain language.

## Step 6: Confirm Design

Before building anything, present a summary table of the complete workflow design:

| # | Property Name | Type | Model | Thinking | Grounded | Purpose |
|---|---------------|------|-------|----------|----------|---------|
| 1 | Document Upload | file | manual input | — | — | User uploads the document |
| 2 | Document Type | single_select | gemini_3_flash | minimal | No | Classify the document |
| ... | ... | ... | ... | ... | ... | ... |

Ask the user: "Does this look right? Any changes before I build it?"

Do NOT proceed to Step 7 until the user explicitly approves the design.

## Step 7: Build via MCP

Use the V7 MCP tools to create the workflow. Follow this sequence:

1. Create the project (agent) with the agreed name
2. Add properties one at a time, in order (later properties may reference earlier ones)
3. After each property is added, report progress: "Added 3/8: Company Name (text, GPT-5, grounded)"

If any MCP call fails, explain what happened in plain language and suggest a fix. Do not show raw error messages to the user.

After all properties are added, confirm: "Your workflow is built. Here is what was created:" followed by a clean summary.

## Step 8: Test

Guide the user to test the workflow:

1. "Upload 3-5 sample documents that represent the types you typically process."
2. Run the workflow on the samples.
3. Review results together:
   - Are extracted values correct? (Check against the source using AI Citations)
   - Are classifications accurate?
   - Are summaries useful and well-formatted?
4. If adjustments are needed, identify which property to modify and update it via MCP.
5. "Once you are happy with the results on 5+ documents, the workflow is ready to scale."

## Troubleshooting

If the user runs into issues during building or testing:

- **Extraction missing data** — Check if `is_grounded` is enabled. Check if the prompt includes N/A handling. Try increasing thinking effort one level.
- **Inconsistent outputs** — Tighten the prompt with explicit formatting rules. Consider switching to a stronger model.
- **Wrong classifications** — Add clearer descriptions for each option in the prompt. Add edge case examples.
- **Slow performance** — Check if a lighter model can handle the task. Lower thinking effort. Replace LLM calls with Python code where possible.
- **High cost** — Bundle related extractions into a single JSON property. Use `gpt_5_mini` or `gemini_3_flash` for simple tasks. Use `code` for formatting and calculations.
