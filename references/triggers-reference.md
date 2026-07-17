# Triggers Reference

Triggers are events that automatically start a workflow run in V7 Go. Instead of manually submitting documents or kicking off an agent, triggers let your workflows respond to things happening in the outside world -- an email arriving, a file landing in SharePoint, a scheduled time firing, or a downstream system requesting data.

Think of triggers as the "when this happens, do that" layer. They turn your agents from on-demand tools into always-on automation.

---

## Three Trigger Types

### 1. Inbound Triggers (Pipedream)

**What they do:** Listen for events from external services and start a workflow run when something happens.

**Examples:**
- A new email arrives in a shared inbox
- A file is uploaded to SharePoint or Google Drive
- A form submission comes in
- A record is created in a CRM

Inbound triggers use **Pipedream connectors** under the hood. Pipedream provides hundreds of pre-built integrations with services like Microsoft 365, Google Workspace, Slack, Salesforce, HubSpot, and many more. When an event fires in one of those services, Pipedream catches it and passes the data to your V7 Go agent.

**Available Pipedream integrations include:**
- Microsoft SharePoint, OneDrive, Outlook
- Google Drive, Gmail, Google Sheets
- Slack, Microsoft Teams
- Salesforce, HubSpot
- Jira, Asana, Monday.com
- Dropbox, Box
- And hundreds more (full list at [Pipedream integrations](https://pipedream.com/apps))

### 2. Time Triggers

**What they do:** Run your workflow on a schedule -- hourly, daily, weekly, or at specific times.

**Examples:**
- Generate a compliance report every Monday at 9 AM
- Pull new submissions from a database every hour
- Run a daily reconciliation check at end of business

Time triggers support two modes:
- **Interval mode** -- run every N minutes or hours
- **Schedule mode** -- run on specific days at specific times

Time triggers are set up through the V7 Go UI: Add Property > Trigger > Time.

### 3. Outbound Triggers (Webhooks)

**What they do:** Send data to an external system when something happens inside your V7 Go workflow.

Unlike inbound triggers (which bring data in), outbound triggers push data out. They fire when events occur on your agent's entities -- for example, when a new entity is created or when all fields finish processing.

**Examples:**
- Notify a Slack channel when a document finishes processing
- Push extracted data to an ERP system when fields complete
- Update a CRM record when an analysis is done

**Available webhook events:**
- `entity.created` -- fires when a new entity is created
- `entity.all_fields_completed` -- fires when all properties finish processing

---

## How Triggers Work: The Backing Project Pattern

This is the most important concept to understand about inbound triggers. When you create an inbound trigger on an agent, V7 Go creates a **backing project** behind the scenes. This backing project is where the raw trigger data actually lands.

Here is the flow:

1. You create an inbound trigger on your agent (the "main project")
2. V7 Go creates a hidden backing project with a `trigger-raw-data` property
3. When the trigger fires, the raw event data goes into the backing project
4. Your main project's properties read from the backing project to access that data

**Why this matters:** If you are building or debugging trigger-based workflows, you need to know that the trigger data lives in the backing project, not in your main agent. You will need to discover the backing project ID and its `trigger-raw-data` property to wire up the data flow correctly.

### Discovering the Backing Project

After creating an inbound trigger:

1. List your agent's properties and find the one with `"type": "trigger"`
2. Read its `config.project_id` -- that is the backing project ID
3. List the backing project's properties to find the `trigger-raw-data` property
4. Use that property's ID as the input for your first processing step

---

## Creating and Deploying Triggers

### Creating an Inbound Trigger

Inbound triggers are created via the API:

```
POST /api/workspaces/{wid}/projects/{project_id}/triggers/inbound
```

With a body like:

```json
{
  "name": "SharePoint: New File Created"
}
```

### Deploying a Trigger

Creating a trigger is not enough -- you must also **deploy** it. Deployment is the step that activates the trigger so it actually starts listening for events.

```
POST /api/workspaces/{wid}/projects/{project_id}/triggers/inbound/{trigger_id}/deploy
```

With a body like:

```json
{
  "target_action": "create_entity",
  "row_splitter": {"type": "none"},
  "property_mappings": []
}
```

The `target_action: "create_entity"` tells V7 Go to create a new entity (a new row in your agent) each time the trigger fires.

### Creating an Outbound Trigger (Webhook)

```
POST /api/workspaces/{wid}/triggers
```

```json
{
  "project_id": "your-agent-uuid",
  "events": {
    "entity.created": true,
    "entity.all_fields_completed": true
  },
  "action": {
    "type": "webhook",
    "url": "https://your-endpoint.com/webhook"
  }
}
```

---

## Common Trigger Patterns

### Email-to-Agent

An email arrives, the agent processes the attachment, and results are available in V7 Go.

1. Create an inbound trigger connected to your email service (Gmail, Outlook)
2. The trigger fires when a new email matches your criteria
3. Your agent's first property extracts the attachment from the trigger data
4. Downstream properties process the document

### Scheduled Report

A report runs automatically on a schedule without anyone needing to kick it off.

1. Add a Time trigger to your agent (daily, weekly, etc.)
2. The agent runs at the scheduled time
3. A skill fetches fresh data from your source system
4. Processing properties generate the report
5. An outbound webhook or skill delivers the report (email, Slack, etc.)

### Webhook Notification

When processing completes, notify an external system.

1. Your agent processes documents (triggered by any method)
2. An outbound trigger listens for `entity.all_fields_completed`
3. When all fields finish, V7 Go sends the results to your webhook URL
4. Your receiving system handles the data (update a database, send a notification, etc.)

---

## Trigger Gotchas

These are hard-won lessons from production V7 Go deployments. Read them before building trigger-based workflows.

### The Backing Project Is Not Auto-Wired (via API)

When you create an inbound trigger through the API, V7 Go creates the backing project but does **not** automatically create a `trigger` system property in your main project. The UI does this automatically, but the API does not. If you are building programmatically, you need to create a `reference` property pointing to the backing project and use it as the `via_property_id` for inputs that need to reach the trigger data.

### Trigger Data Lives in the Backing Project

The raw event data from an inbound trigger is stored in the backing project, not your main project. To access it, you need to:

1. Enable the trigger (`POST /triggers/inbound/{id}/enable`) -- the response includes the `backing_project_id`
2. List that project's properties to find the `Webhook Data` property
3. Reference that property in your main agent's processing chain

### Always Deploy After Setup

Triggers must be explicitly deployed after creation and after all property wiring is complete. If you create a trigger but forget to deploy it, nothing will happen when events fire. The deploy call with `target_action: "create_entity"` is what activates the listener.

### Triggers Are Workspace-Scoped

Inbound triggers (Pipedream connectors) are tied to a specific workspace. If you are migrating agents between workspaces or regions (EU to US), triggers must be created and deployed fresh in the target workspace. They cannot be cloned or transferred.

---

## API Reference

| Action | Endpoint |
|--------|----------|
| Create inbound trigger | [inbound-trigger-create](https://docs.go.v7labs.com/reference/inbound-trigger-create-and-deploy-pipedream-trigger) |
| Enable inbound trigger | [inbound-trigger-enable](https://docs.go.v7labs.com/reference/inbound-trigger-enable) |
| Create outbound trigger | [trigger-create](https://docs.go.v7labs.com/reference/trigger-create) |
| List trigger sources | [trigger-sources-list](https://docs.go.v7labs.com/reference/trigger-sources-list) |
| Triggers documentation | [Triggers](https://docs.go.v7labs.com/docs/triggers) |
