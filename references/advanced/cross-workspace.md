# Advanced Reference: Cross-Workspace Operations

> **This is advanced reference material for solutions engineers and power users. Most V7 Go users won't need this -- start with the main references instead.**

This guide covers working across multiple V7 Go workspaces within a single organization. It applies to enterprise clients with multiple business units, teams, or departments that require information barriers between workspaces -- for example, separate workspaces for different fund teams, compliance vs. operations, or regional divisions.

---

## Chinese Firewall Rules

### 5 Hard Rules for Workspace Isolation

1. **No data leakage between workspaces.** Once a workspace is in production, agents in one workspace must NEVER read, reference, or access entities/hubs/files from another workspace.

2. **Hubs are workspace-scoped.** A hub in one workspace must only contain that team's data. Hub content must not be shared cross-workspace.

3. **Skills that call the V7 Go API must only target their own workspace.** Use `V7Go.metadata["workspace_id"]` at runtime -- never hardcode a cross-workspace ID.

4. **Triggers and webhooks must be workspace-scoped.** Outbound webhooks and inbound triggers must only flow data within the same workspace.

5. **Reference properties** (`type: reference`) can only reference projects within the same workspace.

### Exceptions

- **During migration only:** A staging/technology workspace can read from other workspaces to port agents, hubs, and skills. This is a one-time operation, not an ongoing pattern.
- **Shared marketplace skills:** V7 Go provides organization-level marketplace skills (e.g. `[Slack] List Channels`) that are shared across workspaces. These do not violate firewalls.
- **Generic custom skills** (e.g. `Markdown to DocX`, `Create Shareable File Link`) are duplicated into each workspace automatically. They operate on data within the calling workspace and are safe.

---

## The Staging / Technology Workspace

If the organization has a technology or deployment team, they should have a dedicated staging workspace:

- It is the **only workspace** that may hold configuration, documentation, and references to all other workspaces.
- Local project folders on disk can reference any workspace (this is outside V7 Go).
- The client resources file tracks IDs for ALL workspaces.
- This workspace is used for prototyping, testing, and porting before deploying to team-specific workspaces.

---

## Local Folder Structure

Each project should have its own subfolder under `projects/`, regardless of which workspace it targets. The `project.json` file should include a `workspace_id` and `workspace_name` field to make the target workspace unambiguous:

```json
{
  "name": "Agent Name",
  "workspace_id": "target-workspace-uuid",
  "workspace_name": "Team Name",
  "project_id": "...",
  "description": "..."
}
```

This allows the local file system to serve as an index across all workspaces while maintaining the local-first workflow.

---

## API Key Considerations

- **During migration**, a single API key with access to all workspaces is typical.
- **Post-migration**, consider generating per-workspace API keys for additional isolation.
- **Integration secrets** (e.g. for Front, NetSuite, SharePoint) must be created in each target workspace individually via `PUT /workspaces/{wid}/secrets`. Secrets cannot be read from a source workspace -- values must be known or regenerated.

---

## Concierge Email Addresses

Each workspace has its own Concierge email address:

```
chat+{workspace_id}_{project_id}@agent.v7labs.com
```

When rebuilding agents in new workspaces, Concierge email addresses change. Update any email forwarding rules, distribution lists, or documentation that references old addresses.

---

## Cross-Workspace Gotchas

1. **Property IDs are workspace-scoped.** A property ID from one workspace is meaningless in another. All `@<P{id}>` references in prompts must use the new IDs created in the target workspace.

2. **Hub IDs are workspace-scoped.** Hub references in property descriptions (`@<H{id}>`) and `hub_select` defaults must be updated after migration.

3. **Skill IDs differ between workspaces** (for custom skills). Even identical code recreated in a new workspace gets a new ID. Update all `enabled_skills` and `config.entrypoint_id` references.

4. **Reference properties can only reference projects in the same workspace.** If agent A references agent B, both must exist in the same workspace.

5. **Triggers must be redeployed.** Pipedream triggers are tied to the workspace. New triggers must be created and deployed in each target workspace.

6. **Collection child project IDs change.** When recreating a project with collections, the child project IDs are generated fresh. Any code that hardcodes child project IDs must be updated.

7. **Entity data does not migrate** via property-by-property rebuild. Only agent configuration (properties, views, skills, hubs) is rebuilt. Historical entity data stays in the source workspace. Use the Clone with Data playbook (see `rebuild-playbooks.md`) if entity data must migrate.

8. **Secrets must be created per workspace.** A secret in one workspace is not available in another. Use `PUT /workspaces/{wid}/secrets` to create them in each target workspace.
