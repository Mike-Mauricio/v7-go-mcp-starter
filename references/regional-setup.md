# Regional Setup Guide

V7 Go runs as two **regionally isolated instances** -- EU and US. These are completely separate deployments with separate data stores, separate authentication, and separate API keys. An EU API key cannot access a US workspace, and vice versa.

The platform API (endpoints, payload shapes, property types, behavior) is identical across regions. The only things that change are the **host URL** and the **API key** used to authenticate.

---

## Regional URLs

| | EU Instance | US Instance |
|---|---|---|
| **UI** | `https://go.v7labs.com` | `https://go.us.v7labs.com` |
| **API base** | `https://go.v7labs.com/api/workspaces/{wid}` | `https://go.us.v7labs.com/api/workspaces/{wid}` |
| **MCP URL** | `https://mcp.go.v7labs.com` | `https://mcp.go.us.v7labs.com` |
| **API key env var** | `GO_API_KEY` | `GO_US_API_KEY` |
| **Auth header** | `X-API-KEY: <key>` | `X-API-KEY: <key>` |
| **Docs** | `https://docs.go.v7labs.com` (shared) | Same |

---

## How to Check Which Region You Are In

Look at the **host** in the URL. The workspace UUID does not encode region information.

```
https://go.v7labs.com/{wid}/projects        -->  EU  -->  use GO_API_KEY
https://go.us.v7labs.com/{wid}/projects      -->  US  -->  use GO_US_API_KEY
```

Always derive the region from the host, never from the workspace UUID.

---

## MCP Connection

Add V7 Go as a connector directly in your AI tool's settings — no config files needed.

### EU Instance

1. Open Settings → Connectors → Add Custom Connector
2. Name: **V7 Go**
3. URL: `https://mcp.go.v7labs.com`
4. Complete OAuth login and select your EU workspace

### US Instance

1. Open Settings → Connectors → Add Custom Connector
2. Name: **V7 Go (US)**
3. URL: `https://mcp.go.us.v7labs.com`
4. Complete OAuth login and select your US workspace

V7 Go will appear as a connector in your AI tool once authenticated.

---

## API Keys and Secrets

### Two Keys, Two Regions

Each region requires its own API key. Store both in your `.env` file:

```
GO_API_KEY=<eu-key>
GO_US_API_KEY=<us-key>
```

### Workspace Secrets Do Not Cross Regions

A secret stored in an EU workspace is invisible to a US workspace. When building agents in a new region, every secret a skill depends on must be created in that region's workspace.

### Preflight Check

Before doing any work against a specific region, verify your key is valid:

```bash
# Check EU
curl -s -o /dev/null -w "EU -> HTTP %{http_code}\n" \
  -H "X-API-KEY: $GO_API_KEY" https://go.v7labs.com/api/workspaces

# Check US
curl -s -o /dev/null -w "US -> HTTP %{http_code}\n" \
  -H "X-API-KEY: $GO_US_API_KEY" https://go.us.v7labs.com/api/workspaces
```

`HTTP 200` = key is valid. `HTTP 401` = key is missing or wrong for that region.

---

## Data Residency Rules

EU and US are separate data-residency boundaries. In production:

- **No cross-region data flow.** Do not build skills in one region that read from another region's workspace. Do not set up cross-region webhooks.
- **Skills are region-portable by design.** If a skill reads `V7Go.metadata.__dict__["api_base_url"]` at runtime (instead of hardcoding `go.v7labs.com`), the same code works in either region. The metadata returns the correct host for the region the skill is running in.
- **Never hardcode `go.v7labs.com` in skill code.** Always use the metadata-provided base URL so your skills work in any region.

### What Is Region-Scoped

Everything is region-scoped. These resources exist independently in each region and cannot be shared:

- Workspaces and workspace IDs
- API keys and workspace secrets
- Agents (projects), properties, and property IDs
- Skills and skill IDs
- Hubs, hub files, and hub IDs
- Triggers and trigger backing projects
- Entities and entity data

---

## Cross-Region Migration

If you need to move an agent configuration from one region to another (e.g., EU to US), this is a **full rebuild** -- not a copy or clone. The clone API does not work across regions (it is same-workspace only).

The process involves:
1. Reading the source agent's configuration from the origin region (read-only)
2. Saving definitions locally
3. Recreating every resource (properties, skills, hubs, triggers) in the target region
4. Re-mapping all IDs (property IDs, skill IDs, hub IDs are all new in the target)
5. Recreating secrets in the target workspace
6. Re-uploading hub files and reindexing
7. Deploying triggers fresh in the target workspace

Entity data (historical submissions) does not migrate -- only the agent configuration.

For detailed migration steps, see `advanced/rebuild-playbooks.md`.
