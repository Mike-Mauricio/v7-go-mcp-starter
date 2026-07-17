# Workflow Patterns Guide

This guide covers the most common workflow design patterns in V7 Go, drawn from production builds across financial services, real estate, insurance, and legal verticals. Each pattern includes what it is, when to use it, and a concrete (anonymized) example.

These patterns are building blocks. Most production workflows combine two or more patterns together.

---

## Pattern 1: Sequential Pipeline

**Input -> Classify -> Extract -> Analyze -> Synthesize -> Status**

The most fundamental pattern. Documents flow through a series of steps in order, where each step depends on the output of the previous one.

### How It Works

1. **Input:** A document arrives (uploaded, emailed, or triggered via integration).
2. **Classify:** A single_select property identifies what type of document it is.
3. **Extract:** Text and JSON properties pull out the relevant data fields.
4. **Analyze:** Additional properties evaluate the extracted data — comparing against thresholds, checking for completeness, flagging anomalies.
5. **Synthesize:** A text property generates a summary, memo, or recommendation based on all prior steps.
6. **Status:** A single_select property assigns a final status (approved, needs review, rejected).

### When to Use It

- Standard document processing where each step logically follows the previous one.
- Workflows where classification drives what gets extracted (different document types need different fields).
- Any pipeline where you need a clear, auditable trail from input to output.

### Example: Insurance Submission Intake

A commercial insurance carrier receives submission packets from brokers. Each packet contains a mix of applications, loss runs, and supplemental documents.

| Step | Property Type | Tool | What It Does |
|------|--------------|------|-------------|
| Upload | file | manual | Broker uploads the submission packet. |
| Document Type | single_select | gemini_3_flash | Classifies each document: application, loss run, schedule, supplemental. |
| Key Fields | json | gpt_5 | Extracts insured name, effective dates, coverage lines, limits, and deductibles. |
| Loss History | json | gpt_5 | Pulls loss run data: dates of loss, amounts, claim status. |
| Guideline Check | text | claude_4_5_sonnet | Compares extracted data against underwriting guidelines stored in a Knowledge Hub. |
| Summary | text | claude_4_5_sonnet | Generates a one-page submission summary with key metrics and flags. |
| Triage Status | single_select | claude_4_5_sonnet | Assigns: "Auto-Decline," "Refer to Underwriter," or "Fast Track." |

---

## Pattern 2: Parallel Execution

**Multiple extraction properties running simultaneously once inputs are ready.**

Not every step needs to wait for the previous one. When multiple properties all depend on the same input (the uploaded document), they can run at the same time — dramatically reducing processing time.

### How It Works

1. **Input:** Document arrives.
2. **Parallel extraction:** Multiple text, JSON, or number properties all run simultaneously against the same source document. None of them depend on each other.
3. **Convergence:** A downstream property (summary, analysis, or status) waits for all parallel steps to complete, then synthesizes the results.

### When to Use It

- High-volume processing where speed matters.
- Documents with multiple independent data points (e.g., a financial statement has revenue, expenses, and balance sheet data — each can be extracted independently).
- Any workflow where extraction steps don't depend on each other's output.

### Example: Financial Statement Processing

An investment firm processes quarterly financial statements from portfolio companies. Each statement requires multiple independent extractions.

| Step | Property Type | Tool | Runs In | What It Does |
|------|--------------|------|---------|-------------|
| Upload | file | manual | — | Analyst uploads the financial statement. |
| Revenue Data | json | gemini_3_flash | Parallel | Extracts revenue line items, growth rates, segment breakdowns. |
| Expense Data | json | gemini_3_flash | Parallel | Extracts operating expenses, cost categories, margins. |
| Balance Sheet | json | gemini_3_flash | Parallel | Extracts assets, liabilities, equity, key ratios. |
| Cash Flow | json | gemini_3_flash | Parallel | Extracts operating, investing, and financing cash flows. |
| Key Metrics | number (x5) | code | Parallel | Calculates EBITDA, debt-to-equity, current ratio, etc. from extracted data. |
| Executive Summary | text | claude_4_5_sonnet | After all above | Synthesizes all extractions into a narrative summary with trend analysis. |

The first five extraction steps all run at the same time against the uploaded document. The summary waits for all of them to finish.

---

## Pattern 3: Two-Pass Extraction

**First pass on primary document, second pass on supplementary documents.**

Some workflows need to process a main document first, then use that context to process supporting documents differently.

### How It Works

1. **First pass:** Process the primary document — extract key fields, classify it, pull core data.
2. **Context set:** The outputs from pass one become context for pass two.
3. **Second pass:** Process supplementary documents with awareness of what was found in the primary document.

### When to Use It

- Deal rooms where you have a primary agreement and dozens of supporting exhibits, schedules, and amendments.
- Insurance submissions where the application is primary and loss runs, financials, and supplements are secondary.
- Any situation where understanding the main document changes how you read the supporting ones.

### Example: Commercial Real Estate Lease Review

A real estate fund reviews a new lease package: the primary lease agreement plus exhibits, amendments, and rent schedules.

| Pass | Document | What Happens |
|------|----------|-------------|
| **Pass 1** | Master Lease Agreement | Extract: tenant name, premises, base rent, term, renewal options, key clauses (assignment, subletting, default, insurance requirements). |
| **Pass 2** | Rent Schedule (Exhibit A) | Extract rent escalation details — using base rent from Pass 1 to validate schedule accuracy. |
| **Pass 2** | Amendment #1 | Extract changes — cross-reference against Pass 1 terms to flag what's been modified. |
| **Pass 2** | Estoppel Certificate | Verify tenant-reported terms match the extracted lease terms from Pass 1. |
| **Synthesis** | All outputs | Generate a lease abstract with a discrepancy report highlighting any mismatches between documents. |

---

## Pattern 4: Knowledge Hub-Grounded Analysis

**Querying reference documents before making decisions.**

Instead of relying solely on what the AI model "knows," this pattern connects your workflow to a Knowledge Hub containing your organization's actual guidelines, policies, or standards. The AI reads the reference material and applies it to the document being processed.

### How It Works

1. **Input:** Document arrives for processing.
2. **Extract:** Key data points are pulled from the document.
3. **Hub Query:** A hub_select property connects to a Knowledge Hub containing relevant guidelines.
4. **Grounded Analysis:** A text property uses both the extracted data and the Knowledge Hub context to make a decision or recommendation.
5. **Output:** Decision with explicit references to the guideline sections that informed it.

### When to Use It

- Compliance checking against internal policies or regulatory standards.
- Underwriting decisions based on published guidelines.
- Contract review against standard playbook terms.
- Any workflow where the "right answer" depends on your organization's specific rules — not general knowledge.

### Example: Underwriting Guideline Compliance

A specialty insurer checks every new submission against their published underwriting guidelines before quoting.

| Step | Property Type | Tool | What It Does |
|------|--------------|------|-------------|
| Upload | file | manual | Submission documents uploaded. |
| Extract Submission Data | json | gpt_5 | Pulls: industry class, revenue, loss history, requested limits. |
| Underwriting Guidelines | hub_select | — | Connects to the Knowledge Hub containing the carrier's underwriting manual. |
| Compliance Check | text | claude_4_5_sonnet | Reads extracted data + underwriting guidelines. Reports which guidelines are met, which are not, and which are borderline. Cites specific guideline sections. |
| Risk Score | number | code | Calculates a numeric risk score based on compliance results. |
| Recommendation | single_select | claude_4_5_sonnet | "Within Guidelines," "Refer — Exceptions Needed," or "Decline — Outside Appetite." |

The compliance check property explicitly references the Knowledge Hub content, so every recommendation is grounded in the carrier's actual rules — not the model's general training.

---

## Pattern 5: Code Property for Zero-Cost Logic

**Python for formatting, parsing, calculations — no AI model needed.**

Not everything needs an AI model. Code properties run Python for deterministic tasks: math, string formatting, date calculations, data transformations, conditional logic. They cost nothing (no model inference fees) and produce consistent, repeatable results.

### How It Works

A code property takes inputs from other properties and runs Python code to produce an output. No AI model is invoked — it's pure computation.

### When to Use It

- **Calculations:** Summing line items, computing ratios, calculating percentages.
- **Formatting:** Converting dates, cleaning text, standardizing names.
- **Conditional logic:** "If amount > threshold, set flag to True."
- **Data transformation:** Reshaping JSON, merging arrays, filtering lists.
- **Validation:** Checking that extracted values are within expected ranges.

### Example: Portfolio Risk Scoring

An asset manager extracts financial data from portfolio company reports, then uses code properties to calculate standardized metrics.

| Step | Property Type | Tool | What It Does |
|------|--------------|------|-------------|
| Revenue | number | gemini_3_flash | Extracts total revenue from the financial statement. |
| COGS | number | gemini_3_flash | Extracts cost of goods sold. |
| Total Debt | number | gemini_3_flash | Extracts total debt. |
| Total Equity | number | gemini_3_flash | Extracts total equity. |
| Gross Margin | number | code | `(revenue - cogs) / revenue * 100` |
| Debt-to-Equity | number | code | `total_debt / total_equity` |
| Risk Flag | single_select | code | `"High" if debt_to_equity > 3 else "Medium" if debt_to_equity > 1.5 else "Low"` |
| Formatted Report Date | text | code | Converts extracted date string to standardized `YYYY-MM-DD` format. |

The four code properties add zero cost to the workflow. They run instantly and produce the same result every time — no model variability.

---

## Pattern 6: Multi-Model Pipeline

**Different models for different steps — optimized for accuracy, speed, and cost.**

V7 Go is model-agnostic, which means you can assign different AI models to different properties within the same workflow. This lets you optimize each step for what matters most: fast and cheap models for simple extraction, powerful models for complex analysis.

### How It Works

Each property in a workflow can use a different model. You match the model's strengths to the task:

| Task Type | Best Model Choice | Why |
|-----------|------------------|-----|
| Simple extraction (names, dates, amounts) | Gemini 3 Flash | Fast, cheap, accurate on straightforward fields. |
| Structured JSON extraction | GPT-5 | Strong at following JSON schemas precisely. |
| Nuanced analysis and compliance | Claude 4.5 Sonnet | Excellent at interpreting context, comparing against guidelines. |
| Complex multi-step reasoning | o3 or Claude 4.5 Opus | Best for tasks that require extended chain-of-thought reasoning. |
| Calculations and formatting | Code (Python) | Zero cost, deterministic, instant. |

### When to Use It

- Any workflow with more than 3-4 properties — there's almost always an opportunity to use a faster, cheaper model for the simpler steps.
- High-volume workflows where cost optimization matters.
- Workflows mixing extraction (model-heavy) with calculation (code-only).

### Example: M&A Due Diligence Pipeline

A private equity firm processes due diligence documents for a potential acquisition. The workflow uses four different tools across its properties.

| Step | Property Type | Tool | Why This Tool |
|------|--------------|------|---------------|
| Document Classification | single_select | gemini_3_flash | Fast classification — doesn't need heavy reasoning. |
| Key Terms Extraction | json | gemini_3_flash | Straightforward field extraction from structured documents. |
| Financial Data | json | gpt_5 | Complex table extraction with precise JSON schema adherence. |
| Compliance Review | text | claude_4_5_sonnet | Nuanced analysis comparing terms against fund's investment criteria. Needs strong contextual reasoning. |
| Red Flag Detection | text | o3 | Multi-step reasoning across all extracted data to identify non-obvious risk patterns. |
| Metric Calculations | number (x6) | code | Ratios, growth rates, multiples — pure math, zero cost. |
| Executive Summary | text | claude_4_5_sonnet | Synthesizes all findings into a polished narrative. |
| Deal Score | number | code | Weighted scoring formula based on all extracted metrics and flags. |

This pipeline uses Gemini for speed on simple tasks, GPT-5 for JSON precision, Claude for analytical depth, o3 for complex reasoning, and code for calculations. Each model is playing to its strengths.

---

## Pattern 7: LLM-as-a-Judge (Multi-Model Validation)

Two independent AI models extract the same data from a document. A third "judge" model compares their outputs and resolves disagreements — routing high-confidence results straight through and flagging discrepancies for human review.

### How It Works

```
Document → Extraction A (Model 1) → 
                                      → Judge (Model 3) → STP or Human Review
Document → Extraction B (Model 2) → 
```

1. **Extraction A** — Model from Provider 1 extracts the data
2. **Extraction B** — Model from Provider 2 extracts the same data independently (different provider = different failure modes)
3. **Judge** — A strong reasoning model compares both extractions against the original document, resolves conflicts, and assigns a confidence status
4. **Routing** — Results that pass go straight through (STP); disagreements route to human review

The judge sees the original document — it doesn't just compare the two outputs. It can verify against the source when the extractors disagree.

### When to Use It

- High-stakes extraction where errors are expensive (financial data, legal terms, compliance)
- Workflows processing documents with variable quality (some clean, some messy)
- When you need quantifiable confidence scores for downstream processes
- Regulatory or audit contexts where "two sources agree" adds trust

### Example: Invoice Processing with Dual Verification

A finance team processes vendor invoices with strict accuracy requirements for AP automation.

| Step | Property Type | Tool | Purpose |
|------|--------------|------|---------|
| Invoice Upload | file | manual | User uploads the invoice |
| Document Type | single_select | gemini_3_flash | Classify: standard invoice, credit memo, or debit note |
| Extraction A | json | gpt_5 | Extract: vendor, amount, date, line items, tax |
| Extraction B | json | gemini_2_5_pro | Same extraction, different provider |
| Judge | json | claude_4_5_sonnet | Compare A vs B, verify against source, produce final output |
| Result | single_select | code | "STP" if judge says all_passed, "Needs Review" if not |

**Views:** Create a "Straight-Through" view filtered to Result = STP, and a "Needs Review" view for discrepancies. Enable AI Citations on the Judge property so reviewers can see exactly where the judge found each value.

### Judge Prompt Best Practices

- Give the judge explicit matching rules: text fields use semantic comparison, numbers use a tolerance (e.g., ±$0.01), dates match to the calendar day
- Default to conservative: if unsure, route to human review
- Have the judge produce diagnostics: which fields matched, which didn't, and why
- Enable grounding on the judge property — it should cite the source document

---

## Pattern 8: Multi-Agent Architecture (Master/Child)

A "master" agent programmatically creates and configures "child" agents via the V7 Go API. This enables event-driven systems where one agent can spawn specialized agents for different document types, clients, or workflows.

### How It Works

1. **Master agent** receives an event (trigger, email, API call)
2. Master creates infrastructure: Knowledge Hub, child project, config properties, trigger
3. Master deploys the trigger on the child agent
4. Child agent runs independently, processing its own documents

### When to Use It

- You need to create many similar agents with slight variations (different clients, different document types)
- An event should automatically set up a new processing pipeline
- You're building a platform-like experience where new workflows are provisioned on demand
- Volume requires splitting work across specialized agents

### Example: Client Onboarding Automation

A professional services firm automatically provisions a document processing agent for each new client engagement.

| Step | What Happens |
|------|-------------|
| 1 | New client record created in CRM → triggers master agent |
| 2 | Master creates a Knowledge Hub with the client's reference docs |
| 3 | Master creates a child agent with extraction properties tailored to the client's document types |
| 4 | Master deploys an email trigger on the child agent |
| 5 | Client team forwards documents to the trigger email → child agent processes them |

**Note:** This is an advanced pattern that requires direct API calls. It's designed for solutions engineers building scalable systems, not for typical end-user workflow design. See `references/advanced/` for implementation details.

---

## Combining Patterns

Production workflows almost always combine multiple patterns. Here's how they layer:

| Combination | Example |
|------------|---------|
| Sequential + Parallel | Classify first (sequential), then extract multiple fields simultaneously (parallel), then synthesize (sequential). |
| Two-Pass + Knowledge Hub | Extract from primary document, query Knowledge Hub for guidelines, then process supplementary documents with that context. |
| Multi-Model + Code | Use different AI models for different extraction steps, then code properties for all calculations and formatting. |
| Sequential + Knowledge Hub + Code | Extract data, check against guidelines, calculate a score, assign a status. |

The key principle: **use the simplest pattern that gets the job done.** Start with a sequential pipeline. Add parallelism when speed matters. Add Knowledge Hubs when you need grounded decisions. Add code properties wherever you can replace AI with deterministic logic.
