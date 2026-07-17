# V7 Go Platform Guide

## What Is V7 Go?

V7 Go is an AI agent platform built for automating complex, document-intensive workflows. Think of it as your team's AI-powered document processing engine — it reads, extracts, analyzes, and acts on documents at scale, with built-in proof that every answer traces back to its source.

A few things that make V7 Go distinct:

- **No-code.** You build workflows by configuring properties and connecting steps visually. No programming required.
- **Model-agnostic.** V7 Go works with GPT-5, Claude, Gemini, and other leading AI models. You pick the best model for each step of your workflow — or let V7 choose.
- **Enterprise-grade.** SOC 2 Type II certified, ISO 27001 compliant, GDPR-ready, and HIPAA-ready. Built for industries where accuracy and auditability are non-negotiable.

V7 Go is **not** a chatbot, copilot, or general AI assistant. It is purpose-built for turning messy, unstructured documents into clean, structured, actionable data — with full traceability.

---

## The Five Toolsets

V7 Go gives you five main tools. Each serves a different purpose, and most workflows use several of them together.

### 1. Chats

**What it does:** A conversational interface where you can ask questions about your documents, get summaries, or interact with your Knowledge Hubs.

**When to use it:**
- Quick, one-off questions about a document or set of documents
- Exploring what's in a document before building a full workflow
- Ad hoc analysis where you don't need a repeatable pipeline

**Think of it as:** Asking a knowledgeable colleague to read a document and answer your questions about it.

### 2. Knowledge Hubs

**What it does:** Centralized, searchable libraries where you store reference materials — policy manuals, underwriting guidelines, contract playbooks, compliance standards, pricing sheets. V7 Go's proprietary Index Knowledge technology breaks these large files into searchable indexes for high-accuracy retrieval.

**When to use it:**
- You have reference documents that your AI agents need to consult (e.g., "Does this submission comply with our underwriting guidelines?")
- You want a single source of truth that multiple workflows can query
- You need better accuracy than standard search — Index Knowledge solves the limitations of basic document retrieval

**Think of it as:** A smart filing cabinet that your AI agents can instantly search and reference.

### 3. Integrations

**What it does:** Connects V7 Go to external systems — CRMs, email, cloud storage, spreadsheets, Slack, and more. Supports MCP Connectors (400+ built-in apps), Zapier (8,000+ apps), and a REST API for custom connections.

**When to use it:**
- Triggering workflows from incoming emails or document uploads
- Pushing extracted data into your CRM, ERP, or spreadsheet
- Sending notifications when a workflow completes or needs human review

**Think of it as:** The bridges that connect V7 Go to the rest of your tech stack.

### 4. Agents

**What it does:** Multi-step AI pipelines that chain models and tools together. Agents handle the end-to-end flow: receive a document, classify it, extract data, analyze it against your guidelines, produce output, and route for review. Each agent is a workflow with properties (extraction fields), logic (conditional steps), and integrations.

**When to use it:**
- Repeatable document processing that follows the same steps every time
- Complex workflows with multiple extraction steps, conditional logic, or human review stages
- Any process where you need consistency, speed, and auditability at scale

**Think of it as:** A specialized team member trained on exactly one task, who does it the same way every time, at machine speed.

### 5. Skills

**What it does:** Reusable, modular capabilities that agents can call on. Skills define specific actions — like "extract a table from page 3" or "compare this clause against our standard template." You build a skill once and use it across multiple agents.

**When to use it:**
- You have a common extraction or analysis step that shows up in multiple workflows
- You want to standardize how a particular task is done across your organization
- You're building a library of capabilities that different teams can share

**Think of it as:** Pre-built tools in a toolbox that any of your agents can pick up and use.

---

## How to Choose the Right Tool

Start with this question: **"How many files am I working with?"**

| Scenario | Recommended Tool | Why |
|----------|-----------------|-----|
| 1 file, quick question | **Chat** | Fastest path to an answer. No setup needed. |
| 1 file, structured extraction | **Agent** (single-file workflow) | When you need specific fields extracted consistently. |
| A batch of similar files | **Agent** (batch workflow) | Process 10 or 10,000 files the same way. |
| Reference docs that agents need to consult | **Knowledge Hub** | Store guidelines, policies, or standards that inform decisions. |
| Repeatable sub-tasks across workflows | **Skill** | Build once, reuse everywhere. |
| Connecting to external systems | **Integration** | Get data in and out of V7 Go automatically. |

If you're unsure, start with a Chat to explore your documents, then build an Agent when you know what you need.

---

## Three Workflow Output Patterns

Every V7 Go workflow follows one of three patterns:

### Pattern 1: Unstructured Data to Structured Data

**Input:** Messy, unstructured documents (PDFs, scans, images, contracts, financial statements).
**Output:** Clean, structured data pushed into downstream systems (CRM fields, database rows, spreadsheet columns).

**Example:** A stack of commercial lease PDFs goes in. Clean rows of tenant names, rent amounts, lease terms, and renewal dates come out — ready to load into your portfolio management system.

### Pattern 2: Unstructured Data to Structured Document

**Input:** Raw documents and data sources.
**Output:** A polished, formatted document (Word doc, PDF, Excel report, memo).

**Example:** A collection of due diligence documents goes in. A formatted investment memo comes out — complete with financial summaries, risk flags, and key terms pulled from across all the source documents.

### Pattern 3: Human-AI Augmentation

**Input:** Documents that need both AI processing and human judgment.
**Output:** AI handles 80-90% of the work; humans review, approve, or refine the rest.

**Example:** Insurance submissions are processed by AI (data extraction, risk scoring, guideline checking). Underwriters review flagged items and edge cases — spending their time on judgment calls instead of data entry.

---

## Property Types

Properties are the building blocks of V7 Go workflows. Each property defines a field that gets filled — either by AI, by code, or manually. Here's a quick overview of the types available:

| Property Type | What It Does |
|---------------|-------------|
| **file** | Holds uploaded documents (PDFs, images, spreadsheets). The starting point for most workflows. |
| **text** | Extracts or generates text from documents. The most common property type. |
| **single_select** | Picks one option from a predefined list (e.g., document type, risk level, approval status). |
| **multi_select** | Picks one or more options from a list (e.g., applicable regulations, document categories). |
| **json** | Extracts structured data as JSON — useful for tables, nested data, or complex objects. |
| **collection** | Groups related properties together into a repeatable set (e.g., line items on an invoice). |
| **url** | Captures or generates URLs. |
| **hub_select** | Connects to a Knowledge Hub — lets the workflow reference and query your stored documents. |
| **number** | Extracts or calculates numeric values (amounts, counts, scores). |
| **page_splitter** | Splits multi-page documents into sections for individual processing. |
| **reference** | Links to another workflow or dataset — useful for cross-referencing. |

For a complete reference on each property type, including available tools, grounding support, and configuration details, see [Property Types Reference](./property-types-reference.md).

---

## AI Citations (Grounding)

AI Citations are V7 Go's answer to the question: **"How do I know this is right?"**

When V7 Go extracts a data point from a document, AI Citations trace that answer back to the exact location in the original source — with visual bounding boxes on the PDF showing precisely where the information came from.

### Why This Matters

- **Trust.** Users can verify any extraction with one click. No black-box answers.
- **Compliance.** Regulated industries (finance, insurance, healthcare, legal) need audit trails. AI Citations provide them automatically.
- **Error detection.** When something looks wrong, citations let reviewers instantly see what the AI was reading — and whether it misinterpreted the source.
- **Regulatory alignment.** Supports requirements under SOX, HIPAA, FDA regulations, KYC/AML, and other frameworks that demand traceable, auditable data.

AI Citations are available on text and JSON property types when using supported models. This is one of V7 Go's strongest differentiators — most competing platforms cannot provide this level of source traceability.

---

## Security and Compliance

V7 Go is built for enterprise environments where data security is a requirement, not a nice-to-have.

| Certification / Standard | Status |
|--------------------------|--------|
| **SOC 2 Type II** | Certified |
| **ISO 27001** | Compliant |
| **GDPR** | Ready |
| **HIPAA** | Ready |

V7 Go's security posture is designed for industries handling sensitive data — financial records, medical documents, legal contracts, and personally identifiable information.

---

## Use Cases by Vertical

### Financial Services
**Segments:** Private equity, venture capital, hedge funds, asset managers, endowments.

| Use Case | What V7 Go Does |
|----------|-----------------|
| Data room analysis | Reads and extracts key terms from hundreds of deal documents. |
| Financial statement extraction | Pulls line items, ratios, and figures from income statements, balance sheets, and cash flow statements. |
| M&A due diligence | Processes diligence documents in parallel, flags risks, summarizes findings. |
| Investment thesis building | Synthesizes data from multiple sources into structured investment memos. |
| 10-K / 10-Q analysis | Extracts financial data and management discussion points from SEC filings. |
| Earnings call synthesis | Turns call transcripts into structured summaries with key metrics and guidance. |

### Real Estate
**Segments:** Commercial real estate managers, large funds, property developers.

| Use Case | What V7 Go Does |
|----------|-----------------|
| Lease abstraction | Extracts key terms (rent, term, options, clauses) from commercial leases. |
| Property valuation | Pulls and structures valuation data from appraisal reports. |
| Market analysis | Processes market reports and comp data into structured summaries. |
| Risk evaluation | Analyzes environmental reports, inspection documents, and compliance records. |
| Rent roll analysis | Extracts tenant and financial data from rent rolls across a portfolio. |
| Environmental assessment review | Reads Phase I/II reports and flags key findings. |

### Insurance
**Segments:** Large-form insurers, high document volume carriers, MGAs.

| Use Case | What V7 Go Does |
|----------|-----------------|
| Submission extraction | Pulls data from insurance submissions, slips, and MRCs. |
| Claims automation | Processes claims documents, extracts relevant data, routes for review. |
| Underwriting support | Checks submissions against underwriting guidelines stored in Knowledge Hubs. |
| Actuarial data extraction | Pulls structured data from actuarial reports and loss runs. |
| Policy verification | Compares issued policies against bound terms. |
| Triage and routing | Classifies incoming documents and routes them to the right team. |

### Legal
**Segments:** Law firms, corporate legal departments, contract management teams.

| Use Case | What V7 Go Does |
|----------|-----------------|
| Contract review | Extracts and flags key clauses, obligations, and deadlines. |
| Due diligence | Processes large document sets for M&A, litigation, or regulatory review. |
| Compliance checking | Compares documents against regulatory requirements or internal policies. |
| Document summarization | Turns lengthy legal documents into structured summaries. |

---

## Key Proof Points

When evaluating V7 Go's impact, here are the numbers that matter:

| Metric | Result |
|--------|--------|
| **Accuracy** | 95-99% on one-shot benchmarks — surpasses leading LLMs, RPA/IDP tools, and hyperscaler offerings. |
| **Financial statement processing** | 21x faster with 54% accuracy improvement over manual processes. |
| **Lease abstraction** | 12x faster for CRE lease abstraction and portfolio analysis. |
| **Productivity** | 35% increase in due diligence processes. |
| **Cost savings** | Up to 80% cost reduction on individual document processing workflows. |
| **Time to value** | POC in 7 days, commercial deployment in 11 days from first call. |

---

## Getting Started

The fastest path to your first workflow:

1. **Start with a Chat.** Upload a document and ask questions about it. Get familiar with what V7 Go can extract.
2. **Build a simple Agent.** Pick one document type and one extraction task. Configure your properties and run it on a small batch.
3. **Add a Knowledge Hub.** Upload your reference documents (guidelines, policies, standards) and connect them to your agent.
4. **Connect an Integration.** Wire up the input (email, file upload) and output (CRM, spreadsheet, Slack) to automate the full pipeline.
5. **Scale.** Add more document types, more properties, more agents. Build Skills for common tasks and share them across your team.

For detailed guidance on property configuration, see the [Property Types Reference](./property-types-reference.md).
For workflow design patterns, see the [Workflow Patterns Guide](./workflow-patterns.md).
For connecting to external systems, see the [Integration Guide](./integration-guide.md).
