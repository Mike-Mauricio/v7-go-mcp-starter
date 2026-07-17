# Advanced Reference: Rebuild Playbooks

> **This is advanced reference material for solutions engineers and power users. Most V7 Go users won't need this -- start with the main references instead.**

## What This Document Is For

When you need to recreate V7 Go agents, skills, knowledge hubs, or entire workspaces from scratch, you cannot simply clone them. The clone API is unreliable for standard migration (it produces broken property chains, orphaned references, and fails entirely for cross-workspace scenarios with custom skills). Instead, you rebuild resources property-by-property in a controlled, dependency-aware sequence.

This document provides step-by-step playbooks for each resource type. Each playbook includes Human-in-the-Loop (HITL) checkpoints where manual intervention is required (authentication, secret provisioning, smoke testing). The playbooks are designed to be executed by an LLM agent with API access to both source and target workspaces.

**Who needs this:**
- Solutions engineers migrating client workspaces
- Anyone rebuilding agents after workspace restructuring
- Enterprise deployments requiring workspace isolation between business units

---

## Why Not Use the Clone API

The `POST /projects/{id}/clone` endpoint must never be used for standard agent migration. It:

- Often fails silently, producing agents with broken property chains
- Does not work across workspaces for projects with workspace-specific skills
- Carries over orphaned trigger configurations and stale skill references
- Does not update internal `@<P{id}>` references in property descriptions
- Sometimes duplicates properties incorrectly or loses view assignments
- Fails entirely for cross-workspace cloning when the source has custom skills

The only permitted use of the clone API is Playbook 6 (Clone with Data) when entity data must travel with the agent AND you follow the controlled 3-stage process. For all other cases, use the property-by-property approach in Playbook 1.

---

## Dependency Resolution Order

Resources must be created in this order. A resource that references another must be created after its dependencies exist in the target workspace.

**Level 0: Workspace-Level Resources (create first)**
1. **Secrets** -- API keys, OAuth tokens, integration credentials
   - Upsert via `PUT /workspaces/{wid}/secrets/{secret_name}` (returns HTTP 204)
   - `POST /secrets` is not exposed (404)
   - Secrets cannot be read from the source -- you must know or regenerate the values
2. **Hubs** -- Knowledge bases (Playbook 4)
3. **Custom Skills** -- Reusable Python integrations (Playbook 2)
4. **Agent Skills (CC Skills)** -- Workspace-scoped AI skills (Playbook 5)

**Level 1: Agents / Projects** -- Create the project container

**Level 2: Properties** -- Create in dependency order (bottom-up):
1. File properties (no inputs, manual)
2. Classification properties (input: file only)
3. Extraction properties (input: file, classification)
4. Validation properties (input: extractions)
5. Routing properties (input: validation)
6. Terminal/output properties (input: routing)

**Level 3: Views** -- Create after all properties (views reference property IDs)

**Level 4: Triggers** -- Create last (triggers reference the project and its properties)

---

## ID Translation Map

Maintain a running map of source to target IDs throughout any rebuild:

```python
id_map = {
    # Workspace
    "source_workspace_id": "target_workspace_id",
    # Projects
    "source_project_id": "target_project_id",
    # Properties (add as each is created)
    "source_prop_1": "target_prop_1",
    # Hubs
    "source_hub_1": "target_hub_1",
    # Skills
    "source_skill_1": "target_skill_1",
}

def translate_description(desc, id_map):
    """Replace all @<P{old_id}> and @<H{old_id}> with target IDs."""
    for old_id, new_id in id_map.items():
        desc = desc.replace(old_id, new_id)
    return desc
```

Use this map in Pass 2 to translate all `@<P{id}>` and `@<H{id}>` references in property descriptions, all `inputs` arrays, all `enabled_skills` references, and all `config.entrypoint_id` / `config.hub_id` fields.

### Prioritizing Rebuilds

When determining the order to rebuild agents across a workspace migration:

1. **Self-contained agents first** -- Agents with no hub, skill, or reference dependencies are fastest to rebuild and verify.
2. **Business urgency** -- Which agents are blocking production workflows?
3. **Dependency chains** -- If agents B and C both depend on a shared skill, build the skill first.
4. **Complexity** -- Simple agents (few properties, no collections) before complex ones (many properties, nested collections, triggers).

---

## Playbook 1: Agent Rebuild (Projects)

### Prerequisites

Before starting, ensure the following are already created in the target workspace:
- All hubs the agent references (Playbook 4)
- All custom skills the agent uses (Playbook 2)
- All reference projects the agent links to (rebuild those first)
- All secrets required by skills (Playbook 2, Step 2)

### Step 1: Audit the Source Agent

```
GET /workspaces/{source_wid}/projects/{source_pid}
GET /workspaces/{source_wid}/projects/{source_pid}/properties
GET /workspaces/{source_wid}/projects/{source_pid}/views
```

For each property, extract and record:
- `id`, `name`, `type`, `tool`, `description`, `inputs`, `enabled_skills`, `config`
- `is_grounded`, `tool_config`, `skip_behaviour`, `visible_in_entity_sidebar`
- All `@<P{id}>` references in the description
- All `@<H{id}>` references in the description
- All skill IDs in `enabled_skills`
- Which view(s) include it in their `property_ids`

### Step 2: Build the ID Translation Map

Initialize the map with already-created resources (hubs from Playbook 4, skills from Playbook 2).

### Step 3: Create the Agent

```
POST /workspaces/{target_wid}/projects
{
    "name": source["name"],
    "description": source["description"],
    "icon": source["icon"],
    "icon_color": source["icon_color"]
}
```

Record the new project ID in the id_map.

### Step 4: Create Properties -- Pass 1 (Placeholders)

Process properties in topological order (properties with no inputs first). For each property, create it with `description: "placeholder"`, empty `inputs`, and empty `enabled_skills`.

**Config adaptation rules by type:**

| Type | Rule |
|------|------|
| `single_select` / `multi_select` | Copy `options` array as-is |
| `collection` / `file_collection` | `{"properties": [{"name": "...", "type": "..."}]}` -- V7 Go auto-creates the child project |
| `number` | `{}` -- empty config required (rejects missing config but also rejects `number_type`) |
| `reference` | `{"project_id": translated_id, "entity_limit": N}` -- needs translated reference project ID |
| `trigger` | Cannot be created via the property API. See Step 7 for workaround |
| `json_action` | Omit `entrypoint_id` for now (added in Pass 2) |
| `hub_select` | Omit `hub_id` for now (added in Pass 2) |
| `file`, `json`, `text`, `data` | Omit config entirely (rejected if present) |
| All others | Copy config as-is |

After all properties are created, GET any collection properties to discover their auto-generated `child_project_id`, and recursively rebuild child project properties.

### Step 5: Create Properties -- Pass 2 (Wire Up)

For each property, translate the description (replace all old IDs with new IDs from the map) and reconnect:
- **inputs** -- translate `property_id` values using the id_map
- **enabled_skills** -- translate `skill_id` values
- **config** -- translate any `entrypoint_id`, `hub_id`, `project_id`, or `target_project_id` fields
- **description** -- replace all `@<P{old_id}>` and `@<H{old_id}>` references

Update each property via `PUT /workspaces/{target_wid}/projects/{target_pid}/properties/{prop_id}`.

### Step 6: Create Views

The main view always exists and should be updated. Non-main views must be created. For each view:
- Translate all `property_ids` using the id_map
- Translate all filter `property_id` values
- PUT the main view, POST non-main views

### Step 7: Deploy Triggers (if applicable)

If the source agent has trigger-type properties:

1. Create the inbound trigger via `POST /workspaces/{target_wid}/projects/{target_pid}/triggers/inbound`
2. Enable it via `POST .../triggers/inbound/{trigger_id}/enable` with `{"provider": "other"}` (or `"pipedream"` for OAuth-based triggers)
3. Discover the backing project's `Webhook Data` property
4. Create a reference property as a trigger bridge (see critical gotcha below)
5. Wire trigger data into property inputs in Pass 2

**Critical Gotcha -- Trigger System Property:**
The V7 Go API does NOT auto-create the `trigger` system property (`type: trigger`, `owner: system`) in the main project when using the API flow. This property IS auto-created when using the UI. Without it, properties that reference `Webhook Data` from the backing project via `via_property_id` will show as "unknown input."

**Workaround:** Create a standard `reference` property that points to the trigger's backing project. Use this reference property's ID as the `via_property_id` for any inputs that need to reach `Webhook Data` in the backing project.

**HITL: Trigger Authentication** -- If the trigger uses Pipedream (e.g., SharePoint, Gmail), the user must authenticate via the V7 Go UI after trigger creation.

### Step 8: Verify

- Verify all properties have correct inputs (except file/manual properties)
- Check no orphaned `@<P{old_id}>` references remain
- Verify all views exist and match the source count

**HITL: Smoke Test** -- The user should create a test entity/case and verify the property chain executes correctly.

### Step 9: Update Client Resources

Add the new agent, its properties, and views to the client resources tracking file.

---

## Playbook 2: Custom Skills (Integrations)

Custom integrations are Python skills with `type: "code"` that have source code, dependencies, and may require secrets and domain allowlisting.

### Step 1: Audit the Source Skill

```
GET /workspaces/{source_wid}/skills/{skill_id}
```

Extract: `name`, `description`, `type`, `code`, `dependencies`, `allowed_domains`, `arg_specs` (reveals secret dependencies via `arg_type: "secret"`), `output_type`, `has_kwargs`.

### Step 2: Resolve Secrets

Check `arg_specs` for entries with `arg_type: "secret"`. Each has a `secret_name` field. Verify these secrets exist in the target workspace. If missing, HITL is required -- the user must provide the secret values (they cannot be read from the source workspace).

Create secrets via `PUT /workspaces/{target_wid}/secrets/{SECRET_NAME}` with `{"value": "...", "allowed_domains": [...]}`.

### Step 3: Inspect Code for Hardcoded IDs

Search the skill code for any hardcoded UUIDs:
- Source workspace ID should be replaced with `V7Go.metadata["workspace_id"]`
- Known IDs in the id_map should be translated
- Unknown UUIDs should be flagged for review

### Step 4: Check if Skill Already Exists

Organization-level skills (like `Markdown to DocX`, `Create Shareable File Link`) may already be available in the target workspace. Check before creating duplicates.

### Step 5: Create the Skill

```
POST /workspaces/{target_wid}/skills
{
    "name": ...,
    "description": ...,
    "type": ...,
    "code": ...,  (translated code from Step 3)
    "dependencies": ...,
    "allowed_domains": ...
}
```

### Step 6: Verify

**HITL: Verify Skill Authentication** -- If the skill calls external APIs that require authentication (e.g., Front, SharePoint, NetSuite), the user must test via the V7 Go UI.

---

## Playbook 3: AI Skills (Built-in Tools)

AI Skills are V7 Go's built-in, non-LLM property tools. They are platform-provided and do NOT need to be recreated -- they are available in every workspace automatically.

### Built-in AI Skill Inventory

| Tool Value | Name | What It Does |
|------------|------|--------------|
| `ocr` | OCR | Extracts text from images and scanned documents |
| `pdf_convert` | PDF Convert | Converts documents (e.g., DOCX) to PDF |
| `summary` | Summary | Generates summaries of document content |
| `web_search` | Web Search | Searches the web for information |
| `sandbox_agent` | Sandbox Agent | Runs a sandboxed AI agent for complex tasks |
| `slides_py` | Slides (Python) | Generates PowerPoint/slide presentations |
| `upsert_entity` | Upsert Entity | Creates or updates entities in another project |

### What You DO Need to Do

When rebuilding an agent that uses AI Skills:
1. Set the correct `tool` value on the property (e.g., `"tool": "sandbox_agent"`)
2. Copy the `description` (prompt) -- AI Skills use the description as their instruction set
3. Copy the `tool_config` -- Some AI Skills have configuration
4. Copy `inputs` -- AI Skills depend on upstream properties just like LLM properties
5. Translate `@<P{id}>` and `@<H{id}>` references in the description using the ID map

### Special Considerations

- **`sandbox_agent`**: The description is the full instruction set for the sub-agent. It can reference hubs via `@<H{id}>` -- ensure hub IDs are translated.
- **`upsert_entity`**: Config includes a `target_project_id` that must be translated to the target workspace.
- **`web_search`**: Works identically across workspaces. No special handling needed.
- **`ocr` / `pdf_convert` / `summary`**: Typically used in hub backing projects (prefixed `_Hub:`). When you create a hub (Playbook 4), V7 Go auto-creates the backing project with these tools already configured. Do NOT recreate them.

---

## Playbook 4: Knowledge Hubs Rebuild

Hubs are knowledge bases containing files. Migrating a hub means creating a new hub, downloading all files from the source, uploading them to the target, and reindexing.

### Step 1: Audit the Source Hub

```
GET /workspaces/{source_wid}/hubs/{source_hub_id}
```

Record `name`, `description`, `file_project_id`, `index_status`. List files via the backing project entities.

### Step 2: Create the Target Hub

```
POST /workspaces/{target_wid}/hubs
{
    "name": source_hub["name"],
    "description": source_hub["description"]
}
```

### Step 3: Get Source File Tree with Download URLs

Use `GET /hubs/{hub_id}/files` to retrieve the full folder/file tree. Each file entry includes a signed `url` for download. Walk the tree recursively to collect all files and folders.

### Step 4: Upload Files to Target

**Step 4a:** Upload files using source URLs. The `POST /files` endpoint accepts `source_url` to copy files from any accessible URL (including signed S3 URLs from the source hub).

**Step 4b:** Add uploaded files to the hub via `POST /hubs/{hub_id}/files/add`. The endpoint auto-creates intermediate folders. It will fail if a folder already exists -- avoid calling it twice with the same folder names.

### Step 5: Reindex

```
POST /workspaces/{target_wid}/hubs/{new_hub_id}/reindex
```

**HITL: Verify Hub Indexing** -- The user should verify all files appear, wait for indexing to complete (status: "Indexed"), and try a search query.

### Hub Gotchas

- **Hub backing projects**: V7 Go auto-creates a backing project (prefixed `_Hub:`) with OCR, summary, and content extraction properties. Do NOT recreate these manually.
- **File size limits**: Very large files may fail silently. Check upload responses for errors.
- **Reindex timing**: Indexing large hubs can take several minutes. Hub-dependent properties will fail until indexing completes.
- **Max 1,000 items per hub**: Verify the count before uploading.
- **Folder structure**: Recreate folder structure before uploading files if the source hub uses folders.

---

## Playbook 5: Agent Skills (CC Skills)

Agent Skills (also known as CC Skills) are workspace-scoped AI skill definitions that provide reusable capabilities to Chats/Cases. They have `content` (the full skill definition in markdown/YAML frontmatter), optional `supporting_files`, and metadata.

**Critical rule: NEVER create, edit, or interact with global agent skills (`/api/agent-skills/`). Only use workspace-scoped endpoints (`/api/workspaces/{wid}/agent-skills/`).**

### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/workspaces/{wid}/agent-skills` | List all agent skills in workspace (includes global read-only) |
| `POST` | `/api/workspaces/{wid}/agent-skills` | Create a workspace-scoped agent skill |
| `GET` | `/api/workspaces/{wid}/agent-skills/{id}` | Get a specific agent skill |
| `PUT` | `/api/workspaces/{wid}/agent-skills/{id}` | Update an agent skill |
| `DELETE` | `/api/workspaces/{wid}/agent-skills/{id}` | Delete an agent skill |

### Step 1: Audit the Source Skill

Extract: `id`, `alias`, `name`, `description`, `category`, `content`, `disabled`, `metadata` (title, subtitle, inspiration, tags), `supporting_files` (array of objects with `path` and `content`).

### Step 2: Create in Target Workspace

```
POST /workspaces/{target_wid}/agent-skills
{
    "alias": source_skill["alias"],
    "category": source_skill["category"],
    "content": source_skill["content"],
    "disabled": source_skill["disabled"],
    "metadata": source_skill["metadata"],
    "supporting_files": source_skill["supporting_files"]
}
```

Supporting files are passed inline -- V7 Go stores the file content directly, not as separate uploads.

### Step 3: Verify and Fix Alias

The platform may auto-truncate the `alias`. Compare and fix via PUT if needed.

### Step 4: Verify Content Integrity

Verify that `content`, `category`, `name`, and all `supporting_files` match the source.

### Agent Skill Gotchas

1. **Alias auto-truncation**: The platform may shorten the `alias` on creation. Always verify and update via PUT.
2. **Supporting files are inline**: Stored as `{"path": "...", "content": "..."}` objects directly in the skill payload. No separate file upload needed.
3. **Global vs workspace-scoped**: `GET /workspaces/{wid}/agent-skills` returns both global and workspace-scoped skills. Global skills have `workspace_id: null`. Never modify global skills.
4. **No secret dependencies**: Agent Skills don't have `arg_specs` or secrets like Custom Integrations.
5. **Content may reference workspace-specific data**: If the skill's content contains hardcoded workspace IDs, hub IDs, or project IDs, translate them using the ID map.

---

## Playbook 6: Clone with Data (Entity Migration)

**When to use:** Only when entity data must be migrated verbatim to a new workspace. For structure-only migration, use Playbook 1. For creating new entities with mapped references, use Playbook 1 + entity creation via API.

This playbook uses the clone API in a controlled 3-stage process:

```
Source Agent --[clone within same WS, clone_entities=true]--> TEMP Agent (same WS)
TEMP Agent   --[strip all external references]--> Standalone TEMP
Standalone   --[clone cross-WS, clone_entities=true]--> Target Agent (target WS)
Target Agent --[rebuild stripped dependencies]--> Fully Wired Target
TEMP Agent   --[delete]--> (cleanup)
```

### Step 1: Identify External Dependencies

Before cloning, categorize every external reference:

| Category | How to Detect | Stripping Action |
|----------|---------------|------------------|
| Trigger properties | `prop["type"] == "trigger"` | Delete the property |
| Cross-project inputs | `input["property_id"]` not in project's own property IDs | Remove from `inputs` array |
| Cross-project `@<P{}>` refs | UUIDs in description not in project's property set | Remove from description |
| Custom skill refs | `prop["enabled_skills"]` is non-empty | Set `enabled_skills: []` |
| Reference properties | `prop["type"] == "reference"` in child projects | Delete the property |
| Hub references | `@<H{hub_id}>` in description | Remove from description |

### Step 2: Clone Within Same Workspace

Clone with `clone_entities: true` to create a full copy (properties, views, entities, child projects) within the same workspace.

### Step 3: Disable Auto-Recompute on TEMP

Set `auto_recalculations: false` to prevent property recalculation while stripping references.

### Step 4: Audit TEMP and Build Mapping

Build a map from source property IDs to TEMP property IDs by matching on `slug` or `name`.

### Step 5: Strip External References

For each category from Step 1:
- Delete trigger properties
- Remove cross-project inputs and description references
- Clear custom skill references (`enabled_skills: []`)
- Delete reference properties in child projects
- Remove hub references from descriptions

### Step 6: Verify Standalone

Scan for any remaining external UUIDs. All inputs should reference only internal properties. No external `@<P{}>` references should remain. No `enabled_skills` should be present.

### Step 7: Clone Cross-Workspace

Clone the standalone TEMP with `clone_entities: true` into the target workspace.

### Step 8: Disable Auto-Recompute on Target

### Step 9: Audit Target and Build Full ID Map

Build a complete mapping: source to TEMP to target. Match by slug/name.

### Step 10: Rebuild Stripped Dependencies

Using the ID map and the dependency record from Step 1:
- Create inbound trigger (if applicable)
- Re-wire cross-project inputs with target workspace IDs
- Re-add custom skill references
- Recreate reference properties in child projects
- Restore hub references with target hub IDs

### Step 11: Enable Auto-Recompute

**HITL: Smoke Test** -- Verify entity count matches source, check data integrity, confirm rebuilt dependencies work.

### Step 12: Cleanup

Delete the TEMP agent from the source workspace.

### Clone with Data Gotchas

1. **Large projects may fail.** The clone API is not designed for large source projects. Consider cloning in batches or using property-by-property rebuild with manual entity migration.
2. **Collection child projects are cloned too.** Strip and rebuild external references within child projects as well.
3. **Trigger properties cannot survive cross-workspace clone.** Always delete before cross-workspace clone and recreate in the target.
4. **Entity values for stripped properties will be null.** They recompute once dependencies are rebuilt and auto-recompute is enabled.
5. **View filters survive the clone** with auto-translated property IDs. Verify post-clone.
6. **Turn off auto-recompute BEFORE stripping.** If left on, deleting a property may trigger cascading recomputation errors.

---

## HITL Checkpoint Reference

Summary of all Human-in-the-Loop checkpoints across playbooks:

| HITL Step | Playbook | When | Why |
|-----------|----------|------|-----|
| Add Secrets | 2 (Skills) | Before skill creation | Secret values cannot be read from the source workspace |
| Trigger Authentication | 1 (Agents) | After trigger creation | Pipedream OAuth triggers require user authentication via V7 Go UI |
| Verify Skill Authentication | 2 (Skills) | After skill creation | Skills calling external APIs may need OAuth re-authentication |
| Review Sandbox Agent Prompts | 3 (AI Skills) | After agent rebuild | Long sandbox_agent prompts may contain domain-specific references |
| Verify Hub Indexing | 4 (Hubs) | After hub file upload | Indexing is async and must complete before hub-dependent properties work |
| Smoke Test | 1 (Agents) | After full agent rebuild | End-to-end verification of property chain execution |
| Smoke Test (Clone) | 6 (Clone with Data) | After dependency rebuild | Entity data integrity and rebuilt dependency verification |

### HITL Prompt Format

Required checkpoints use:
```
HITL REQUIRED: {Step Name}
{Description of what needs to happen}
{Specific instructions with URLs, steps, etc.}
Confirm when done.
```

Optional/recommended checkpoints use:
```
HITL RECOMMENDED: {Step Name}
{Description of what could be reviewed}
{What to look for}
Would you like to proceed, or skip this check?
```

---

## Full Workspace Rebuild -- Execution Order

When rebuilding multiple resources for a workspace, follow this order:

1. **Secrets** -- HITL: user provides values
2. **Hubs** -- Playbook 4: download, upload, reindex
3. **Custom Skills** -- Playbook 2: code, secrets, domains
4. **Agent Skills** -- Playbook 5: content, supporting files, alias
5. **Reference Agents** -- Playbook 1: agents that other agents reference
6. **Main Agents** -- Playbook 1: full property chain rebuild
7. **Triggers** -- Part of Playbook 1, Step 7: deploy last
8. **Clone with Data** -- Playbook 6: when entity data must migrate
9. **Smoke Tests** -- HITL: user verifies end-to-end

This ensures every dependency exists before anything that references it is created.
