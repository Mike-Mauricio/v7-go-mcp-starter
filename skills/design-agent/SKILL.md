---
name: design-agent
description: >-
  Plan a V7 Go workflow without building it. Produces a readable spec sheet
  with inputs, steps, outputs, models, and prompts. Use when the user wants
  to plan before building, or needs to share a design with their team.
argument-hint: "[description of what you want to automate]"
---

# Design Agent

You are helping a non-technical user plan a V7 Go workflow. This skill produces a readable spec sheet — a document they can review, share with their team, or hand off for building later.

**This skill does NOT build anything.** No MCP calls, no API calls, no deployment. Planning only.

## Step 1: Guided Interview

Ask these questions **one at a time** — wait for each answer before asking the next:

1. "What documents or data do you work with?" (PDFs, scans, spreadsheets, emails, etc.)
2. "What do you currently do with them manually?" (extract data, review for compliance, generate a report, classify and route, etc.)
3. "What should the output look like?" (a data table, a formatted document, a classification/routing decision)
4. "Where does the output need to go?" (stay in V7 Go, push to Slack/email/CRM, download as Excel/PDF)
5. "Who will use this workflow?" (one person, a team, multiple departments)
6. "How many documents do you process per week/month?" (helps gauge scale and cost)

Summarize what you heard back to the user in 2-3 sentences before moving on.

## Step 2: Design the Workflow

Based on the interview answers, design the complete workflow:

### 2a. Choose the Output Pattern

| Pattern | Description | Fits When |
|---------|-------------|-----------|
| **Structured Data** | Documents in, organized data table out | The user wants rows and columns of extracted data |
| **Document Generation** | Documents in, new document out | The user wants formatted memos, reports, or summaries |
| **Human-AI Augmentation** | AI handles 80-90%, humans review the rest | The user needs quality control or exception handling |

### 2b. Map the Pipeline

Design properties following the standard sequence:

```
Input -> Classify -> Extract -> Analyze -> Synthesize -> Status
```

For each property, determine:
- **Name** — Clear, descriptive label
- **Type** — file, text, single_select, multi_select, json, collection, number, url, hub_select
- **Model** — Which AI model powers it (read `references/model-selection-guide.md`)
- **Thinking Effort** — minimal, low, medium, or high
- **Grounded** — Yes for extraction from documents, No for summaries/classifications/generation
- **Prompt Summary** — 1-2 sentence description of what the prompt does

### 2c. Draft Prompts

Write a full prompt for each AI-powered property using the patterns in `references/prompt-patterns.md`. Apply these rules:

- Assign a specific expert role
- Include N/A handling for every extraction prompt
- Specify output format (Markdown, JSON, plain text, bullets)
- Reference inputs with `@<Pproperty_ID>` syntax (use placeholder IDs like `@<Pproperty_INPUT>`)
- Add exclusions — what the model should NOT do
- Include formatting rules — dates (YYYY-MM-DD), numbers, currency

### 2d. Estimate Cost Profile

Based on model choices, provide a rough cost profile:

| Profile | Characteristics | Typical Models |
|---------|----------------|----------------|
| **Light** | Mostly Gemini Flash / GPT-5 Mini, minimal thinking effort, few properties | Low cost per document |
| **Medium** | Mix of models, low-to-medium thinking effort, 5-10 properties | Moderate cost per document |
| **Heavy** | Claude or GPT-5.1 with medium-high thinking, many properties, hub searches | Higher cost per document |

## Step 3: Produce the Spec Sheet

Generate a clean, readable markdown document with the following sections. Use the exact structure below:

```markdown
# [Agent Name] — Workflow Spec

## Agent Overview

| Field | Details |
|-------|---------|
| **Name** | [Agent name] |
| **Purpose** | [One sentence: what this agent automates] |
| **Output Pattern** | [Structured Data / Document Generation / Human-AI Augmentation] |
| **Estimated Cost Profile** | [Light / Medium / Heavy] |

### What It Automates
[2-3 sentences describing the manual process this replaces and the value it provides]

## Inputs

| Property | Type | Description |
|----------|------|-------------|
| [Name] | [Type] | [What the user provides] |

## Processing Steps

| # | Property | Type | Model | Thinking | Grounded | What It Does |
|---|----------|------|-------|----------|----------|--------------|
| 1 | [Name] | [Type] | [Model] | [Effort] | [Yes/No] | [1-sentence description] |
| 2 | ... | ... | ... | ... | ... | ... |

## Prompts

### [Property 1 Name]
**Model:** [model] | **Thinking:** [effort] | **Grounded:** [yes/no]

```
[Full prompt text]
```

### [Property 2 Name]
...

## Outputs

| Output | Format | Description |
|--------|--------|-------------|
| [What the user gets] | [Table / Document / Classification] | [How it is used] |

## Integration Points

| Destination | Method | Details |
|-------------|--------|---------|
| [Where output goes] | [MCP Connector / Zapier / API / Manual] | [Specifics] |

## Notes & Recommendations

- [Any design decisions, trade-offs, or suggestions for the user]
- [Recommended number of test documents before scaling]
- [Any Knowledge Hub setup needed]
```

## Step 4: Save the Spec

Write the spec sheet to a file in the current directory:

- Filename: `[agent-name]-spec.md` (lowercase, hyphens for spaces)
- Example: `lease-abstraction-agent-spec.md`

Tell the user: "Your spec is saved to `[filename]`. You can share this with your team for review, or when you are ready to build, use the build-workflow skill to deploy it."
