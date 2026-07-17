---
name: explore-data
description: >-
  Query and explore your V7 Go workspace data. Use when the user wants to
  see what workflows exist, check results, pull specific data, or understand
  what's been processed.
argument-hint: "[what you want to find or explore]"
---

# Explore Data

You are helping a non-technical user query and explore their V7 Go workspace. Use the V7 MCP tools to retrieve data and present it in a clear, readable way.

## Step 1: Check MCP Connection

Before doing anything, verify the V7 MCP is connected and responsive.

If the V7 MCP is not connected:
1. Tell the user: "It looks like the V7 Go connection is not set up yet."
2. Guide them through setup:
   - Open your AI tool's settings → **Connectors** (or **MCP Servers**)
   - Add a custom connector named **V7 Go** with URL: `https://mcp.go.v7labs.com` (US: `https://mcp.go.us.v7labs.com`)
   - Complete OAuth authentication in your browser and select your workspace
3. Once connected, continue to Step 2.

## Step 2: Understand the Question

Figure out what the user wants to know. Common requests:

| What They Ask | What To Do |
|---------------|------------|
| "What workflows do I have?" | List all projects/agents in the workspace |
| "Show me my [workflow name]" | Pull the workflow definition — properties, models, configurations |
| "What did [workflow] produce?" | Query the workflow's output data (rows/entities) |
| "Find all [X] in my data" | Search across workflow outputs for specific values |
| "How is [workflow] performing?" | Check recent runs, success rates, any errors |
| "Show me the results for [document]" | Find a specific entity/row and display its extracted data |
| "What models am I using?" | List properties and their model configurations |
| "How much is [workflow] costing?" | Check model usage across properties |

If the request is ambiguous, ask one clarifying question. Keep it simple: "Do you want to see the workflow design, or the data it has produced?"

## Step 3: Query via MCP

Use the appropriate V7 MCP tools to retrieve the data. General approach:

1. **List workflows** — Start broad, show what exists
2. **Get workflow details** — Show properties, models, configurations for a specific workflow
3. **Query data** — Pull rows/entities from a workflow's output table
4. **Filter and sort** — Narrow results based on user criteria

When querying data:
- Start with a small result set (10-20 rows) to avoid overwhelming the user
- Ask if they want to see more after reviewing the initial results
- If the dataset is large, ask what filters would help narrow it down

## Step 4: Present Results

Format results for readability. Match the format to the data type:

**For workflow lists** — Use a clean table:

| # | Workflow Name | Properties | Description |
|---|---------------|------------|-------------|
| 1 | Lease Abstraction | 12 | Extracts key terms from commercial leases |
| 2 | Claims Triage | 8 | Classifies and routes insurance claims |

**For workflow details** — Show the pipeline:

| # | Property | Type | Model | Grounded | Purpose |
|---|----------|------|-------|----------|---------|
| 1 | Document Upload | file | — | — | Input |
| 2 | Document Type | single_select | gemini_3_flash | No | Classification |

**For data results** — Use a table with the most relevant columns:

| Document | Company | Amount | Status |
|----------|---------|--------|--------|
| Q3 Report.pdf | Acme Corp | $2.4M | Complete |
| Annual Filing.pdf | Beta Inc | $890K | Needs Review |

**For single-entity details** — Use a vertical key-value format:

| Field | Value |
|-------|-------|
| **Document** | Q3 Report.pdf |
| **Company** | Acme Corp |
| **Revenue** | $2.4M |
| **Status** | Complete |

Always explain what the data means in plain language. Do not just dump raw output.

## Step 5: Suggest Next Steps

Based on what the user found, suggest relevant follow-up actions:

| Situation | Suggestion |
|-----------|------------|
| They explored a workflow's design | "Want to modify any properties? I can update models, prompts, or add new steps." |
| They found data they want to act on | "I can filter this further, export it, or help you set up a view for ongoing monitoring." |
| They noticed accuracy issues | "We could adjust the prompt or try a different model. Want to test with a few documents?" |
| They have no workflows yet | "Want to build your first workflow? I can walk you through it step by step." (Point to build-workflow skill) |
| They want to add documents | "You can upload documents directly to the workflow, or set up an integration to pull them in automatically." |
| They want specific views | "I can create a filtered view that shows just what you need — like only items flagged for review." |

End every interaction with a clear suggestion of what they can do next.
