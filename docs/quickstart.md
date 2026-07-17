# Quickstart: Your First V7 Go Workflow

This guide walks you through building a simple document extraction workflow in about 5 minutes. By the end, you'll have a working V7 Go agent that pulls structured data from your documents.

---

## Step 1: Connect to V7 Go (2 minutes)

If you haven't already:

1. Copy the MCP config: `cp .mcp.json.example .mcp.json`
2. Edit `.mcp.json` if you're in the US region (change URL to `https://mcp.go.us.v7labs.com`)
3. Open the repo in your AI tool (Claude Code, Cursor, or Codex)
4. When prompted, authenticate with your V7 Go credentials in the browser
5. Select your workspace

**How to know it's working:** Ask your AI assistant "Can you list my V7 Go workflows?" If it returns results (or says your workspace is empty), you're connected.

---

## Step 2: Choose a Document (30 seconds)

Pick one document you work with regularly. Good candidates:
- A contract or agreement (PDF)
- A financial statement or report
- An insurance form or application
- A lease or property document

Have the file ready to upload, or describe it to your AI assistant.

---

## Step 3: Tell the AI What You Need (1 minute)

Describe what you want to extract. Be specific about the data points. For example:

> "I have PDF contracts. I need to extract: company name, contract date, contract value, and expiration date. Put the results in a table."

Or:

> "I work with quarterly earnings reports. I need: company name, revenue, net income, and EPS from each report."

Your AI assistant will ask follow-up questions to understand your needs.

---

## Step 4: Review the Design (1 minute)

Your AI assistant will propose a workflow design — something like:

| # | Property | Type | Model | Purpose |
|---|----------|------|-------|---------|
| 1 | Document | file | manual | Upload your PDF |
| 2 | Company Name | text | Gemini 3 Flash | Extract the company name |
| 3 | Contract Date | text | Gemini 3 Flash | Extract the contract date |
| 4 | Contract Value | text | Gemini 3 Flash | Extract the contract value |
| 5 | Expiration Date | text | Gemini 3 Flash | Extract the expiration date |
| 6 | Status | single_select | GPT-5 Mini | Mark as Complete or Needs Review |

Review the properties. Ask to add, remove, or change anything. Once it looks right, confirm.

---

## Step 5: Build and Test (1 minute)

Your AI assistant creates the workflow in your V7 Go workspace using the MCP. You'll see progress updates:

```
Created project: Contract Data Extractor
Added 1/6: Document (file, manual)
Added 2/6: Company Name (text, Gemini 3 Flash)
Added 3/6: Contract Date (text, Gemini 3 Flash)
...
Done! 6/6 properties created.
```

Then test it:
1. Open your V7 Go workspace
2. Find the new workflow
3. Upload a sample document
4. Check the extracted values against the source

The AI Citations (visual bounding boxes) show exactly where each value was found in the document.

---

## What's Next?

Now that you have a working workflow, you can:

- **Add more properties** — Extract additional data points
- **Add a Knowledge Hub** — Reference a guidelines document for compliance checks
- **Set up integrations** — Push extracted data to Google Sheets, Salesforce, Slack, etc.
- **Build a more complex workflow** — Ask your AI assistant: "Help me build a workflow for [your use case]"

### Useful Resources

- [Best Practices](best-practices.md) — 10 design principles for accurate workflows
- [Prompt Patterns](../references/prompt-patterns.md) — Templates for writing effective prompts
- [Model Selection](../references/model-selection-guide.md) — Which AI model for which task
- [FAQ](faq.md) — Common questions and troubleshooting
