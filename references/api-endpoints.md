# V7 Go API Endpoint Reference

This is a quick reference for all V7 Go API endpoints. Most users interact with V7 Go through the MCP — these endpoints are for advanced users who need direct API access.

**Base URLs:**
- EU: `https://go.v7labs.com/api`
- US: `https://go.us.v7labs.com/api`

**Authentication:** `X-API-Key: {your_api_key}` header on every request.

**Documentation tip:** Append `.md` to any V7 Go docs URL to get the full OpenAPI spec in Markdown format.

---

## Projects (Agents/Workflows)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/projects` | List all projects |
| POST | `/workspaces/{ws_id}/projects` | Create a new project |
| GET | `/workspaces/{ws_id}/projects/{project_id}` | Get project details |
| PUT | `/workspaces/{ws_id}/projects/{project_id}` | Update project |
| DELETE | `/workspaces/{ws_id}/projects/{project_id}` | Delete project |
| POST | `/workspaces/{ws_id}/projects/{project_id}/clone` | Clone project (⚠️ use with caution — see gotchas G42) |

## Properties

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/projects/{project_id}/properties` | List properties |
| POST | `/workspaces/{ws_id}/projects/{project_id}/properties` | Add a property |
| GET | `/workspaces/{ws_id}/projects/{project_id}/properties/{prop_id}` | Get property |
| PUT | `/workspaces/{ws_id}/projects/{project_id}/properties/{prop_id}` | Update property (⚠️ full payload required — see gotchas G8) |
| DELETE | `/workspaces/{ws_id}/projects/{project_id}/properties/{prop_id}` | Delete property |

## Entities (Rows/Items)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/projects/{project_id}/entities` | List entities (paginated) |
| POST | `/workspaces/{ws_id}/projects/{project_id}/entities` | Create entity |
| GET | `/workspaces/{ws_id}/projects/{project_id}/entities/{entity_id}` | Get entity |
| PUT | `/workspaces/{ws_id}/projects/{project_id}/entities/{entity_id}` | Update entity |
| DELETE | `/workspaces/{ws_id}/projects/{project_id}/entities/{entity_id}` | Delete entity |

## Skills

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/skills` | List custom skills |
| POST | `/workspaces/{ws_id}/skills` | Create custom skill |
| GET | `/workspaces/{ws_id}/skills/{skill_id}` | Get skill details |
| PUT | `/workspaces/{ws_id}/skills/{skill_id}` | Update skill |
| DELETE | `/workspaces/{ws_id}/skills/{skill_id}` | Delete skill |
| POST | `/workspaces/{ws_id}/skills/{skill_id}/runs` | Execute skill directly |

## Agent Skills (Claude Code-Style Skills)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/agent-skills` | List agent skills |
| POST | `/workspaces/{ws_id}/agent-skills` | Create agent skill |
| GET | `/workspaces/{ws_id}/agent-skills/{skill_id}` | Get agent skill |
| PUT | `/workspaces/{ws_id}/agent-skills/{skill_id}` | Update agent skill |

## Knowledge Hubs

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/hubs` | List hubs |
| POST | `/workspaces/{ws_id}/hubs` | Create hub |
| GET | `/workspaces/{ws_id}/hubs/{hub_id}` | Get hub details |
| PUT | `/workspaces/{ws_id}/hubs/{hub_id}` | Update hub (rename only) |
| DELETE | `/workspaces/{ws_id}/hubs/{hub_id}` | Delete hub |
| GET | `/workspaces/{ws_id}/hubs/{hub_id}/files` | List hub files |
| POST | `/workspaces/{ws_id}/hubs/{hub_id}/files/add` | Add files to hub |
| DELETE | `/workspaces/{ws_id}/hubs/{hub_id}/files/{file_id}` | Remove file from hub |
| POST | `/workspaces/{ws_id}/hubs/{hub_id}/reindex` | Reindex hub |

## Triggers

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/projects/{project_id}/triggers` | List triggers |
| POST | `/workspaces/{ws_id}/projects/{project_id}/triggers` | Create trigger |
| POST | `/workspaces/{ws_id}/projects/{project_id}/triggers/{trigger_id}/deploy` | Deploy trigger |
| DELETE | `/workspaces/{ws_id}/projects/{project_id}/triggers/{trigger_id}` | Delete trigger |

## Folders

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/folders` | List folders |
| POST | `/workspaces/{ws_id}/folders` | Create folder |
| PUT | `/workspaces/{ws_id}/folders/{folder_id}` | Update folder |
| DELETE | `/workspaces/{ws_id}/folders/{folder_id}` | Delete folder |

## Secrets

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/workspaces/{ws_id}/secrets` | List secrets |
| POST | `/workspaces/{ws_id}/secrets` | Create secret |
| DELETE | `/workspaces/{ws_id}/secrets/{secret_id}` | Delete secret |

## File Upload

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/workspaces/{ws_id}/files` | Upload file (via source_url) |
| POST | `/workspaces/{ws_id}/files/start_upload` | Start direct file upload |
| POST | `/workspaces/{ws_id}/files/{file_id}/confirm` | Confirm file upload |

## Agent Builder (AI-Assisted)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/workspaces/{ws_id}/agent_builder` | Start AI agent builder session |
| GET | `/workspaces/{ws_id}/agent_builder/{request_id}` | Poll builder status |
| POST | `/workspaces/{ws_id}/agent_builder/{request_id}/execute` | Execute builder plan |
| POST | `/workspaces/{ws_id}/agent_builder/{request_id}/followup` | Request plan edits |

---

## Full API Documentation

For complete endpoint documentation with request/response schemas, visit:
- [V7 Go API Docs](https://docs.go.v7labs.com/reference)
- Append `.md` to any docs URL for the full OpenAPI spec
