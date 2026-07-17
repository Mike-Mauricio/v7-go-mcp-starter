---
name: getting-started
description: >-
  First-time setup and onboarding for V7 Go with MCP. Use when a new user
  needs help connecting to V7 Go, understanding the platform, or building
  their first workflow.
---

# Getting Started

You are onboarding a new user to V7 Go through their AI coding tool (Claude Code, Cursor, Codex, or similar). Walk them through setup, show them what is possible, and help them get a quick first win.

Use plain language throughout. The user is not a developer — they are a business user who wants to automate document-heavy work.

## Step 1: Welcome

Start with a brief introduction:

"Welcome to V7 Go. You are now connected to an AI agent platform that can automate your document-heavy workflows — extracting data from PDFs, classifying documents, generating reports, and more.

Here is what you can do through this tool:
- **Build workflows** that process documents automatically
- **Query your workspace** to see what has been built and what data exists
- **Run workflows** on documents right from here
- **Manage Knowledge Hubs** — upload reference documents that your workflows can search

Let's get you set up."

## Step 2: Check Setup

Verify the V7 MCP connection is working.

### If V7 Go is not connected:

Tell the user:

"First, we need to connect to your V7 Go workspace. Here's how:"

1. "Open your AI tool's settings and go to **Connectors** (or **MCP Servers**)."
2. "Add a new custom connector named **V7 Go**."
3. "Paste the MCP URL: `https://mcp.go.v7labs.com` (or `https://mcp.go.us.v7labs.com` for US region)."
4. "Complete the OAuth login in your browser and select your workspace."
5. "Once connected, come back here and say 'ready' — I'll verify the connection."

### If the MCP connector exists but is not responding:

Tell the user:

"V7 Go appears to be configured but the connection isn't active. Try these steps:"

1. "Check your connector settings — make sure V7 Go is listed and enabled."
2. "Try disconnecting and reconnecting the V7 Go connector."
3. "If it still doesn't connect, check that your V7 Go account is active and you have workspace access."

### If the MCP is connected:

Tell the user: "Your V7 Go connection is active. Let's see what you have."

## Step 3: Explore the Workspace

Use the V7 MCP to show the user what already exists in their workspace:

1. List all workflows (agents) in the workspace.
2. Present them in a clean summary:

| # | Workflow | Properties | Description |
|---|----------|------------|-------------|
| 1 | [Name] | [Count] | [What it does, in plain language] |

If workflows exist:
- "You already have some workflows set up. Want to explore any of these, or build something new?"

If the workspace is empty:
- "Your workspace is fresh — no workflows yet. Let's build your first one. It will take about 5 minutes."

## Step 4: First Quick Win

Guide the user through building a minimal workflow to experience V7 Go in action. This should be fast and satisfying — 3 properties, one sample document.

### 4a. Identify the Document

Ask: "What document do you work with most often? For example: invoices, contracts, reports, applications, claims, leases."

Based on their answer, suggest 2-3 fields to extract. Examples:

| Document Type | Suggested Fields |
|---------------|-----------------|
| Invoice | Vendor Name, Invoice Total, Due Date |
| Contract | Parties, Effective Date, Term Length |
| Report | Company Name, Report Date, Key Metrics |
| Application | Applicant Name, Submission Date, Status |
| Claim | Claimant Name, Claim Amount, Date of Loss |
| Lease | Tenant Name, Lease Start Date, Monthly Rent |

### 4b. Build a Minimal Workflow

Create a workflow with exactly 3 properties:

1. **File Input** — `file` type, manual input. Where the user uploads their document.
2. **Field 1 Extraction** — `text` type, appropriate model (default: `gemini_3_flash`, thinking: `low`, grounded: `true`). Extracts the first suggested field.
3. **Field 2 Extraction** — `text` type, same model configuration. Extracts the second suggested field.

Optionally add a 4th property:
4. **Status** — `single_select` type with options like "Complete" and "Needs Review". Lightweight classification model (`gpt_5_mini`, thinking: `minimal`).

Before building, show the user the plan:

"Here is what I will build:

| # | Property | Type | What It Does |
|---|----------|------|--------------|
| 1 | Document Upload | file | You upload your document here |
| 2 | [Field 1] | text | Extracts [field 1] from the document |
| 3 | [Field 2] | text | Extracts [field 2] from the document |

Sound good?"

Build it via MCP after the user confirms. Report progress on each property.

### 4c. Run It

1. "Upload a sample document to test it."
2. Trigger the workflow on the uploaded document.
3. Show the results in a clean format.
4. Highlight AI Citations: "See those highlighted sections in your document? Those are AI Citations — they show exactly where each value came from. This is how V7 Go provides proof for every extraction."

### 4d. Celebrate the Win

"Your first workflow is live. It just did in seconds what used to take you [estimated time]. Now imagine running this on hundreds of documents."

## Step 5: What's Next

Point the user to their options:

"Here is what you can do from here:

- **Add more fields** — I can add more extraction steps to this workflow, or more complex analysis like summaries and compliance checks.
- **Build a full workflow** — Use the build-workflow skill for a guided, step-by-step process to create a complete multi-step agent. Just say 'build workflow' followed by what you want to automate.
- **Explore your data** — Use the explore-data skill to query and browse what your workflows have produced.
- **Set up a Knowledge Hub** — Upload reference documents (policy manuals, guidelines, playbooks) that your workflows can search for context.
- **Connect to other tools** — V7 Go integrates with 400+ apps through MCP Connectors, plus 8,000+ through Zapier.

For a deeper understanding of how V7 Go works, check these reference files:
- `references/model-selection-guide.md` — Which AI models to use for which tasks
- `references/prompt-patterns.md` — Reusable prompt templates for common tasks

What would you like to do next?"
