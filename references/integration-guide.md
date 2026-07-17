# Integration Guide

V7 Go connects to external systems in four ways: MCP Connectors (built-in), Zapier, API, and direct integrations. This guide covers how each method works and the most common patterns for wiring V7 Go into your existing tech stack.

---

## Integration Methods at a Glance

| Method | App Coverage | Best For | Setup Complexity |
|--------|-------------|----------|-----------------|
| [MCP Connectors](#mcp-connectors) | 400+ built-in apps | Direct connections to popular tools (CRMs, email, storage, messaging) | Low — toggle on in Settings |
| [Zapier](#zapier) | 8,000+ apps | Connecting to niche or specialized tools not covered by MCP | Low-Medium — Zapier account required |
| [API](#api) | Unlimited (custom) | Custom integrations, internal systems, advanced automation | Higher — requires technical setup |

---

## MCP Connectors

MCP (Model Context Protocol) Connectors are V7 Go's built-in integration layer. They provide direct, native connections to popular business applications — no middleware or third-party accounts needed.

### How to Enable MCP Connectors

1. Go to **Settings** in V7 Go.
2. Navigate to **Integrations**.
3. Find the connector you want (e.g., HubSpot, Salesforce, Google Drive, Slack).
4. Toggle it **on**.
5. Authenticate with your credentials for that service.

That's it. Once enabled, the connector is available across all your workflows.

### Popular MCP Connectors

| Category | Available Connectors | Common Use Cases |
|----------|---------------------|-----------------|
| **CRM** | HubSpot, Salesforce | Push extracted data into contact/deal records. Pull account context into workflows. |
| **Email** | Gmail, Outlook | Trigger workflows from incoming emails. Send processed results via email. |
| **Cloud Storage** | Google Drive, Dropbox, OneDrive, SharePoint, Box | Pull documents from shared drives. Save processed outputs back to storage. |
| **Messaging** | Slack, Microsoft Teams | Send notifications when workflows complete. Alert teams when human review is needed. |
| **Spreadsheets** | Google Sheets, Excel Online | Export extracted data to spreadsheets. Pull reference data from sheets. |
| **Project Management** | Jira, Asana, Monday.com | Create tasks from workflow outputs. Update project tracking with processed data. |
| **Database** | Airtable, Notion | Push structured data to database records. Sync workflow outputs with team databases. |

### Using MCP Connectors in Workflows

MCP Connectors can be added directly to workflow properties, giving your AI agents the ability to read from or write to external systems as part of their processing.

**How to add an MCP tool to a property:**

1. Open your workflow in V7 Go.
2. Select the property where you want the integration to act.
3. In the property's tool configuration, you'll see available MCP tools alongside the AI models.
4. Select the MCP tool you want (e.g., "Send Slack message" or "Create HubSpot contact").
5. Configure the connection details (which channel, which object type, which fields to map).

**Permission auto-approval for workflow properties:** When MCP tools are used within workflow properties (as opposed to interactive chat), permissions are automatically approved. This means the workflow runs end-to-end without pausing for manual permission clicks — essential for automated, high-volume pipelines.

This is an important distinction:
- **In Chat:** MCP tool actions may prompt you to approve the action before it executes (e.g., "V7 Go wants to create a Salesforce record. Allow?").
- **In Workflows:** MCP tool actions execute automatically as part of the pipeline. No manual approval needed.

---

## Zapier

Zapier extends V7 Go's reach to over 8,000 applications. If an app isn't available as a native MCP Connector, Zapier almost certainly supports it.

### How It Works

Zapier connects V7 Go to external apps through **Zaps** — automated workflows with a trigger and one or more actions.

**V7 Go as a Trigger (data flows out):**
- A workflow completes processing -> Zapier detects it -> Zapier sends the output to another app.
- Example: V7 Go extracts invoice data -> Zapier creates a row in QuickBooks.

**V7 Go as an Action (data flows in):**
- Something happens in another app -> Zapier detects it -> Zapier sends data to V7 Go to start a workflow.
- Example: A new email arrives in Gmail with an attachment -> Zapier sends the attachment to V7 Go for processing.

### Common Zapier Triggers (V7 Go -> External App)

| Trigger | What Happens | Example Action |
|---------|-------------|----------------|
| Workflow completes | A document finishes processing in V7 Go. | Create a record in Salesforce with the extracted data. |
| Status changes | A workflow property (like approval status) changes value. | Send a Slack notification to the review team. |
| New output available | Extracted data or a generated document is ready. | Upload the output file to Google Drive. |

### Common Zapier Actions (External App -> V7 Go)

| Trigger (External) | What Happens | V7 Go Action |
|--------------------|-------------|-------------|
| New email received | An email with an attachment arrives. | Upload the attachment to V7 Go and start a workflow. |
| New file in folder | A document is added to a Google Drive folder. | Send the file to V7 Go for processing. |
| Form submitted | A web form is completed. | Create a new workflow run in V7 Go with the form data. |
| Scheduled time | A daily or weekly schedule fires. | Trigger a batch processing workflow in V7 Go. |

### Setting Up a Zap

1. Create a [Zapier](https://zapier.com) account if you don't have one.
2. Search for "V7 Go" in Zapier's app directory.
3. Choose your trigger app and event.
4. Choose your action app and event.
5. Map the fields — tell Zapier which V7 Go outputs go into which fields in the destination app.
6. Test and activate.

---

## API

V7 Go provides a REST API for custom integrations. This is the most flexible option — if you can write (or have someone write) API calls, you can connect V7 Go to anything.

### What the API Can Do

| Capability | Description |
|-----------|-------------|
| **Start workflows** | Programmatically trigger a workflow run with uploaded documents. |
| **Retrieve results** | Pull extracted data, generated documents, and status from completed workflows. |
| **Manage documents** | Upload, download, and organize documents in V7 Go. |
| **Query status** | Check whether a workflow run is in progress, completed, or needs review. |
| **Webhook callbacks** | Receive notifications when workflows complete — V7 Go calls your system. |

### When to Use the API

- **Internal systems:** Your organization has custom software (ERP, proprietary database, internal tools) that needs to send documents to V7 Go or receive processed data.
- **High-volume automation:** Programmatic document submission at scale — hundreds or thousands of files per day.
- **Custom dashboards:** Building a client-facing or internal dashboard that displays V7 Go processing results.
- **Advanced orchestration:** Coordinating V7 Go with other AI/ML services, data pipelines, or business logic that goes beyond what Zapier supports.

### Getting Started with the API

1. Generate an API key in **Settings** -> **API**.
2. Review the API documentation (available in your V7 Go account).
3. Start with a simple test: upload a document and retrieve results.
4. Build from there — add webhook callbacks, batch processing, and field mapping.

If you don't have a technical team, your V7 Go account manager can help scope API integration requirements and connect you with implementation support.

---

## Common Integration Patterns

These are the most frequently used integration patterns across V7 Go deployments. Each combines V7 Go's processing capabilities with external systems for end-to-end automation.

### Pattern 1: Email Trigger -> Extraction -> CRM Push

**The scenario:** Documents arrive via email and need to be processed and logged in your CRM.

**How it flows:**

```
Incoming email with attachment
    |
    v
V7 Go receives the document (via Zapier or MCP email connector)
    |
    v
Workflow extracts key data (names, dates, amounts, terms)
    |
    v
Extracted data is pushed to CRM (HubSpot, Salesforce via MCP connector)
    |
    v
Confirmation notification sent via Slack or email
```

**Real-world example:** A broker sends a submission to an insurer's intake email. V7 Go extracts the insured's name, coverage lines, and requested limits, then creates a new opportunity in Salesforce with those fields pre-populated. The underwriting team gets a Slack message that a new submission is ready for review.

**Integration tools used:** Gmail/Outlook (MCP or Zapier trigger) + V7 Go workflow + Salesforce (MCP connector) + Slack (MCP connector).

---

### Pattern 2: Document Upload -> Extraction -> Spreadsheet Export

**The scenario:** A batch of documents needs to be processed and the results exported to a spreadsheet for analysis or reporting.

**How it flows:**

```
Documents uploaded to V7 Go (manually or from cloud storage)
    |
    v
Workflow processes each document (classify, extract, analyze)
    |
    v
Extracted data exported to Google Sheets or Excel
    |
    v
Team reviews and works with the data in their familiar spreadsheet environment
```

**Real-world example:** A real estate fund uploads 150 commercial leases. V7 Go extracts tenant name, square footage, base rent, lease term, and renewal options from each one. The results populate a Google Sheet that the portfolio management team uses for their quarterly rent roll analysis.

**Integration tools used:** Google Drive (MCP connector for file pickup) + V7 Go workflow + Google Sheets (MCP connector for data export).

---

### Pattern 3: Scheduled Trigger -> Knowledge Hub Query -> Notification

**The scenario:** On a recurring schedule, V7 Go checks something against your Knowledge Hub and notifies the relevant team.

**How it flows:**

```
Scheduled trigger fires (daily, weekly, or custom)
    |
    v
V7 Go queries the Knowledge Hub for updated or relevant content
    |
    v
Results are analyzed against predefined criteria
    |
    v
Notification sent via Slack, email, or Teams with findings
```

**Real-world example:** Every Monday morning, V7 Go checks the compliance Knowledge Hub for any regulatory updates published in the past week. If new guidance is found, it generates a summary and sends it to the compliance team's Slack channel with links to the relevant sections.

**Integration tools used:** Zapier (scheduled trigger) + V7 Go workflow with Knowledge Hub + Slack (MCP connector for notification).

---

### Pattern 4: Document Processing -> Human Review -> Approved Data Push

**The scenario:** Documents are processed by AI, but a human needs to review and approve the results before they're sent to a downstream system. This is the human-in-the-loop pattern.

**How it flows:**

```
Document arrives and is processed by V7 Go workflow
    |
    v
AI extracts data, runs analysis, assigns a confidence score
    |
    v
Results flagged for human review (either all results or only low-confidence ones)
    |
    v
Reviewer approves, edits, or rejects in V7 Go's review interface
    |
    v
Approved data is pushed to the downstream system (CRM, database, ERP)
    |
    v
Rejected items are flagged for re-processing or manual handling
```

**Real-world example:** An insurance carrier uses V7 Go to process claims documents. The AI extracts claim details, calculates reserves, and checks policy coverage. Claims above a certain dollar threshold or with low confidence scores are routed to an adjuster for review. Once the adjuster approves (or edits) the extraction, the approved data is pushed to the carrier's claims management system.

**Integration tools used:** V7 Go workflow (with human review stage) + Claims management system (API or MCP connector) + Slack (MCP connector for reviewer notifications).

---

## Choosing the Right Integration Method

| Question | If Yes | If No |
|----------|--------|-------|
| Is the app available as an MCP Connector? | Use MCP — it's the simplest option. | Check Zapier or API. |
| Do you need the integration inside a workflow property? | Use MCP Connectors (they can be added directly to properties). | Any method works. |
| Is the app available in Zapier? | Use Zapier — no code required. | Use the API. |
| Do you have a technical team? | API gives you the most flexibility. | Stick with MCP or Zapier. |
| Is this high-volume (hundreds+ of documents/day)? | API is most reliable at scale. | MCP or Zapier work well. |

When in doubt, start with MCP Connectors. They're built in, require the least setup, and work directly within your workflows.

---

## Concierge (Email-to-Chats)

V7 Go provides a built-in email integration called **Concierge** that lets users send documents directly to an agent via email — no triggers, no API, no setup.

### How It Works

1. Every V7 Go workspace has a Concierge email address in this format:
   ```
   chat+{workspace_id}_{project_id}@agent.v7labs.com
   ```
2. When someone sends an email with attachments to this address, V7 Go:
   - Creates a new chat session
   - Uploads the attachments
   - Processes them through the agent's workflow

### Agent Design Requirement

For Concierge to work, the agent **must** have a `file` property named exactly **"File"**. This is where the email attachment is deposited.

### When to Use Concierge vs. Triggers

| Feature | Concierge | Email Trigger |
|---------|-----------|--------------|
| Setup required | None — just send an email | Configure trigger + deploy |
| Processing model | Chat-based (conversational) | Workflow-based (structured) |
| Best for | Ad-hoc document processing, one-off requests | High-volume, automated pipelines |
| Output | Chat response | AI Table rows |

**Tip:** Concierge is great for getting started — users can forward documents without any V7 Go training. Migrate to triggers when volume increases or you need structured output.
