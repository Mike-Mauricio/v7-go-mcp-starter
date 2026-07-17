# Property Types Reference

Properties are the building blocks of every V7 Go workflow. Each property defines a single field — something you want to extract, calculate, classify, or collect from your documents. This reference covers every property type, the tools (AI models and other options) available for each, and key configuration details.

---

## Property Types at a Glance

| Type | Purpose | AI Grounding | Typical Use |
|------|---------|:------------:|-------------|
| [file](#file) | Upload and hold documents | N/A | Workflow input — PDFs, images, spreadsheets |
| [text](#text) | Extract or generate text | Yes | Summaries, extracted clauses, analysis |
| [single_select](#single_select) | Pick one option from a list | No | Document classification, status, risk level |
| [multi_select](#multi_select) | Pick one or more options | No | Tags, categories, applicable regulations |
| [json](#json) | Extract structured data | Yes | Tables, line items, nested objects |
| [collection](#collection) | Group repeatable property sets | N/A | Invoice line items, lease provisions |
| [url](#url) | Capture or generate URLs | No | Links, references |
| [datetime](#datetime) | Extract or set dates and times | No | Due dates, effective dates, timestamps |
| [data](#data) | Raw file/data input | N/A | Manual data uploads, raw file inputs |
| [hub_select](#hub_select) | Connect to a Knowledge Hub | N/A | Querying reference documents |
| [number](#number) | Extract or calculate numbers | No | Amounts, scores, counts |
| [page_splitter](#page_splitter) | Split documents into sections | N/A | Breaking multi-page PDFs into parts (UI only — not available through MCP) |
| [reference](#reference) | Link to another workflow | N/A | Cross-referencing datasets |

---

## Detailed Property Type Reference

### file

**What it is:** The entry point for most workflows. A file property holds uploaded documents — PDFs, images, spreadsheets, Word documents, and other file types.

**When to use it:** Every workflow that processes documents starts with at least one file property. This is where your source material lives.

**Available tools:** `manual` (user uploads the file directly).

**Grounding (AI Citations):** Not applicable — file properties hold documents, they don't extract from them.

**Configuration notes:**
- File properties can accept single files or batches.
- Supported formats include PDF, PNG, JPG, TIFF, DOCX, XLSX, and more.
- File properties are typically the first step in a workflow — other properties read from them.

---

### text

**What it is:** The most common property type. Text properties extract, summarize, or generate text from your documents using AI models.

**When to use it:** Anytime you need to pull text from a document — a clause from a contract, a summary of findings, an analysis of a financial statement, a generated memo section.

**Available tools:** `manual`, all AI models (see [Available Tools](#available-tools) below), `code`, `web_search`.

**Grounding (AI Citations):** **Yes.** When using supported AI models, text properties can show exactly where in the source document each piece of extracted text came from — with visual bounding boxes on the original PDF.

**Configuration notes:**
- Write clear, specific prompts. The more precise your instruction, the better the extraction.
- Use grounding whenever accuracy and auditability matter (compliance, financial data, legal terms).
- Text properties can reference other properties in their prompts — e.g., "Based on the document type identified in {classification}, extract the relevant terms."

---

### single_select

**What it is:** A dropdown that picks exactly one option from a predefined list. The AI reads the document and selects the most appropriate choice.

**When to use it:** Document classification (is this a lease, amendment, or rider?), risk levels (low, medium, high), approval status (approved, rejected, needs review), or any field where there's exactly one right answer from a fixed set.

**Available tools:** `manual`, all AI models, `code`.

**Grounding (AI Citations):** **No.** Single select properties return a category choice, not extracted text, so grounding does not apply.

**Configuration notes:**
- Define your options clearly. Each option has a label and a color.
- Available colors: `rainbow-01` through `rainbow-17` (see [Single Select Colors](#single-select-color-options) below).
- Keep option lists focused. If you have more than 10-15 options, consider whether the task could be split into two classification steps.

---

### multi_select

**What it is:** Like single_select, but allows choosing one or more options from the list. The AI can tag a document with multiple applicable categories.

**When to use it:** Tagging documents with multiple attributes — applicable regulations (HIPAA and SOX), document categories (financial and legal), risk factors (credit risk and market risk), or any field where multiple options can apply simultaneously.

**Available tools:** `manual`, all AI models, `code`.

**Grounding (AI Citations):** **No.** Multi select properties return category choices, not extracted text.

**Configuration notes:**
- Same color options as single_select (`rainbow-01` through `rainbow-17`).
- Useful for tagging and filtering workflows — e.g., "Show me all documents tagged with both 'compliance' and 'high priority'."

---

### json

**What it is:** Extracts structured data in JSON format — tables, nested objects, arrays of items. This is how you pull complex, multi-field data from documents.

**When to use it:** Extracting tables (financial line items, rent rolls, policy schedules), nested data structures (a list of parties with their roles and contact info), or any situation where you need more than a single text field.

**Available tools:** `manual`, all AI models, `code`.

**Grounding (AI Citations):** **Yes.** JSON properties support grounding, so you can trace each extracted data point back to its source location in the document.

**Configuration notes:**
- Define your JSON schema clearly in the prompt. Specify the exact fields, data types, and structure you expect.
- JSON properties are powerful for table extraction — define the column headers and let the AI fill in the rows.
- Output can be pushed directly to spreadsheets, databases, or APIs via integrations.

---

### collection

**What it is:** A container that groups related properties into a repeatable set. Think of it as a "row" template — each instance of the collection is one row, and the properties inside it are the columns.

**When to use it:** Invoice line items (each line has a description, quantity, unit price, total), lease provisions (each provision has a clause type, text, and page reference), or any repeating structure within a document.

**Available tools:** Collections themselves don't have tools — the properties inside them do.

**Grounding (AI Citations):** Not directly applicable. The individual properties within a collection can support grounding if they are text or JSON types.

**Configuration notes:**
- Define the properties inside the collection first, then the collection groups them.
- Collections are essential for any document that contains repeating items (invoices, schedules, tables with variable row counts).

---

### url

**What it is:** Captures or generates a URL. Can be extracted from a document or constructed by code.

**When to use it:** Linking to source documents, generating links to external systems, capturing URLs mentioned in documents.

**Available tools:** `manual`, all AI models, `code`.

**Grounding (AI Citations):** **No.**

**Configuration notes:**
- URL properties are often used in integration steps — e.g., generating a link to the processed document's location in your CRM.

---

### datetime

**What it is:** Captures or extracts a date and/or time value. Returns a structured date/time object rather than free-form text.

**When to use it:** Effective dates, due dates, filing deadlines, report dates, timestamps — any field where you need a proper date value rather than a text string.

**Available tools:** `manual`, all AI models, `code`.

**Grounding (AI Citations):** **No.** Datetime properties return a date value, not source-linked text.

**Configuration notes:**
- Datetime properties produce structured date/time values that can be used for sorting, filtering, and calculations.
- For extracted dates, include format instructions in the prompt: "Extract the effective date."
- Use datetime instead of text when you need to sort or filter by date — text dates don't sort correctly.

---

### data

**What it is:** A raw file or data input property. Accepts uploaded files or raw data without AI processing.

**When to use it:** When you need to accept a raw data file as input — CSV uploads, JSON data files, or other structured data that will be processed by downstream properties.

**Available tools:** `manual` (user uploads or provides the data directly).

**Grounding (AI Citations):** Not applicable — data properties hold raw inputs.

**Configuration notes:**
- Data properties are input-only — they accept raw file/data uploads.
- Use data properties when you need a secondary data input alongside the primary file property.

---

### hub_select

**What it is:** Connects a workflow property to a Knowledge Hub. When a hub_select property is configured, the workflow can query your stored reference documents as part of its processing.

**When to use it:** Any workflow that needs to check extracted data against reference material — "Does this submission meet our underwriting guidelines?" or "Which policy section covers this claim type?"

**Available tools:** Configuration only — you select which Knowledge Hub to connect.

**Grounding (AI Citations):** Not directly applicable. The Knowledge Hub query results are used by downstream properties, which may support grounding.

**Configuration notes:**
- You must have a Knowledge Hub created and populated before connecting it via hub_select.
- Hub_select properties are typically placed early in a workflow so that downstream steps can reference the retrieved context.

---

### number

**What it is:** Extracts or calculates a numeric value — dollar amounts, percentages, counts, scores, quantities.

**When to use it:** Financial figures (total revenue, premium amount), calculated metrics (risk score, accuracy percentage), counts (number of pages, number of line items).

**Available tools:** `manual`, all AI models, `code`.

**Grounding (AI Citations):** **No.** Number properties return a numeric value, not source-linked text.

**Configuration notes:**
- For calculated values, use the `code` tool with a Python expression.
- Number properties work well with downstream logic — e.g., "If risk_score > 7, route to senior underwriter."

---

### page_splitter

**What it is:** Splits a multi-page document into individual pages or sections so each can be processed separately.

**When to use it:** When you have a large document (e.g., a 200-page contract or a stack of invoices in a single PDF) and need to process each page or section independently.

**Available tools:** Configuration-based — you define the splitting rules.

**Grounding (AI Citations):** Not applicable.

**Important:** Page splitter properties are **not available through the MCP**. They can only be created and configured through the V7 Go UI. If a workflow needs page splitting, guide the user to set it up in the V7 Go interface.

**Configuration notes:**
- Page splitters are typically placed right after the file input property.
- Useful for batch-in-one-file scenarios (a single PDF containing 50 invoices that each need individual extraction).

---

### reference

**What it is:** Links to another workflow or dataset within V7 Go. Allows one workflow to reference the outputs of another.

**When to use it:** Cross-referencing — e.g., a claims workflow that references the output of a policy verification workflow, or a diligence workflow that pulls in data from a separate financial extraction workflow.

**Available tools:** Configuration-based.

**Grounding (AI Citations):** Not applicable.

**Configuration notes:**
- Reference properties create connections between workflows, enabling multi-stage pipelines.

---

## Available Tools

Every property that supports AI processing lets you choose which tool (model or method) runs it. Here's the full list:

| Tool Name | Type | Notes |
|-----------|------|-------|
| `manual` | Human input | User fills in the value directly. No AI involved. |
| `gpt_5_5` | AI Model | OpenAI GPT-5.5. Latest high-capability GPT. Strong at structured JSON and hub search. |
| `gpt_5_mini` | AI Model | OpenAI GPT-5 Mini. Fast and low cost. Good for classification and simple extraction. |
| `gemini_3_1_pro` | AI Model | Google Gemini 3.1 Pro. High-capability Gemini. Strong at narrative and long-context analysis. |
| `gemini_3_flash` | AI Model | Google Gemini 3 Flash. Fast, cost-effective. Great for high-volume extraction and summaries. |
| `gemini_3_1_flash_image` | AI Model | Google Gemini 3.1 Flash Image. Image generation model. |
| `claude_4_7_opus` | AI Model | Anthropic Claude 4.7 Opus. Most capable Claude model. Best for deep reasoning and complex code. |
| `claude_4_6_sonnet` | AI Model | Anthropic Claude 4.6 Sonnet. Strong at nuanced analysis, compliance, and document synthesis. |
| `claude_4_5_haiku` | AI Model | Anthropic Claude 4.5 Haiku. Fast, low-cost Claude. Good for classification and simple tasks. |
| `auto_llm` | AI Model | Platform default model selection. Available but may produce unpredictable model choices — prefer selecting a specific model. |
| `code` | Code execution | Runs Python code. Zero AI cost — use for formatting, calculations, parsing. |
| `web_search` | Web search | Searches the web for information. Useful for market data, company lookups. |

### Choosing the Right Model

A few rules of thumb:

- **High-volume, straightforward extraction** (names, dates, amounts from clear documents): Use `gemini_3_flash`, `gpt_5_mini`, or `claude_4_5_haiku` for speed and cost efficiency.
- **Complex analysis or nuanced text** (contract clause interpretation, risk assessment): Use `claude_4_6_sonnet` or `gpt_5_5`.
- **Multi-step reasoning** (comparing terms across documents, applying complex rules): Use `claude_4_7_opus` or `gpt_5_5` with high thinking effort.
- **Formatting, math, parsing** (no AI needed): Use `code` — it's free and deterministic.

---

## Thinking Effort Levels

Some models support "thinking effort" — a setting that controls how much reasoning the model does before answering. Higher effort means more thorough analysis but slower and more expensive. Lower effort is faster and cheaper for straightforward tasks.

| Level | Description | Model Compatibility |
|-------|-------------|-------------------|
| **disabled** | No extended thinking. Fastest, lowest cost. | `claude_4_7_opus`, `claude_4_6_sonnet` |
| **minimal** | Very light reasoning. | `gpt_5_mini`, `gemini_3_flash` |
| **low** | Light reasoning for simple tasks. | `gpt_5_mini`, `gemini_3_flash` |
| **medium** | Moderate reasoning. Good default for most tasks. | `gpt_5_mini`, `gemini_3_flash`, `claude_4_7_opus`, `claude_4_6_sonnet` |
| **high** | Thorough reasoning. Use for complex extractions. | `gpt_5_mini`, `gemini_3_flash`, `claude_4_7_opus`, `claude_4_6_sonnet` |

For `gpt_5_5`, `gemini_3_1_pro`, and `claude_4_5_haiku`: Check V7 Go docs for current thinking effort compatibility.

### Important Compatibility Notes

- **Claude Sonnet/Opus** do **not** support `low` thinking effort. Claude jumps from `disabled` directly to `medium`. If you need light reasoning with Claude, use `medium`.
- **GPT-5 Mini** and **Gemini 3 Flash** support `minimal` as their lowest thinking level.
- **GPT-5.5**, **Gemini 3.1 Pro**, and **Claude 4.5 Haiku** are newer models — check V7 Go docs for their current thinking effort support.

---

## Single Select Color Options

When configuring single_select or multi_select properties, each option can be assigned a color. V7 Go uses a rainbow palette with 17 colors:

| Color Code | Visual |
|------------|--------|
| `rainbow-01` | Red |
| `rainbow-02` | Orange-red |
| `rainbow-03` | Orange |
| `rainbow-04` | Amber |
| `rainbow-05` | Yellow |
| `rainbow-06` | Yellow-green |
| `rainbow-07` | Green |
| `rainbow-08` | Teal |
| `rainbow-09` | Cyan |
| `rainbow-10` | Light blue |
| `rainbow-11` | Blue |
| `rainbow-12` | Indigo |
| `rainbow-13` | Purple |
| `rainbow-14` | Violet |
| `rainbow-15` | Magenta |
| `rainbow-16` | Pink |
| `rainbow-17` | Rose |

Use colors intentionally — for example, `rainbow-01` (red) for high risk, `rainbow-05` (yellow) for medium, `rainbow-07` (green) for low. Consistent color use across workflows helps users scan results quickly.
