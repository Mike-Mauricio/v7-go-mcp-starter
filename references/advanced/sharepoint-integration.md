# Advanced Reference: SharePoint Integration

> **This is advanced reference material for solutions engineers and power users. Most V7 Go users won't need this -- start with the main references instead.**

This guide is for connecting V7 Go to Microsoft SharePoint document libraries. It covers authentication, architecture patterns, and production-tested solutions for enumerating, syncing, and processing SharePoint files through V7 Go agents.

---

## Quick Decision Guide

| Question | Recommendation |
|----------|----------------|
| One-off file lookup from an existing folder? | Pipedream `sharepoint-list-files-in-folder` via the OAuth flow. Zero setup. |
| Need to enumerate >200 folders or recurse deeply? | Azure AD client credentials + raw Microsoft Graph (Pipedream caps at ~200 items per call). |
| Continuous cache of a whole drive (1k+ folders)? | The two-agent delta-sync architecture (SP Folder Store + SP Sync Orchestrator). |
| Tens of thousands of files, needs reindexing? | Hub-per-folder architecture (sketched but not built; see Future Patterns below). |
| Quick visibility into folder names without waiting? | Shallow Seed Folders skill -- populates Folder Store with metadata-only entries in ~90 seconds for 2,000+ folders. |

---

## Authentication Options

V7 Go can authenticate to SharePoint two ways. Picking the right one matters because each has different capabilities, scale limits, and configuration cost.

### Option A: Pipedream OAuth

The user connects their SharePoint account once via Pipedream's OAuth flow in the V7 Go UI. The skill gets a `PipedreamAuth` runtime object with a pre-authenticated `session` and an `auth_provision_id`.

**Pros:**
- No Azure AD app registration required. Self-serve through the V7 Go UI.
- Inherits the connected user's permissions.

**Cons / limits:**
- Pipedream's `sharepoint-list-files-in-folder` component silently caps results at ~200 items per call regardless of `$top`. This is the single biggest reason to move to Azure AD for large drives.
- All input props go through Pipedream's `resolve_prop` "search by name" step, which fails for raw Graph IDs (composite site IDs, drive IDs starting with `b!`, folder/item IDs).
- Triggers (e.g. `sharepoint-new-file-created`) are immutable after deployment.

**When to use:** Small/medium folders, named-lookup workflows, prototypes, or when the user wants to wire up their own SharePoint without IT involvement.

### Option B: Azure AD Client Credentials (Raw Microsoft Graph)

The skill calls the Microsoft token endpoint with client credentials (`MS_TENANT_ID`, `MS_CLIENT_ID`, `MS_CLIENT_SECRET` workspace secrets) and gets a bearer token for Microsoft Graph. Then issues normal HTTP requests against `https://graph.microsoft.com/v1.0/...`.

**Pros:**
- No 200-item cap. Full pagination via `@odata.nextLink` works as documented.
- Full Graph API surface -- `$select`, `$orderby`, `$top=500`, delta query, search endpoints.
- Tenant-wide permissions -- same skill works against any folder/drive/site.

**Cons:**
- Requires Azure AD app registration with `Files.Read.All` + `Sites.Read.All` (and `Files.ReadWrite.All` if you need to write).
- Three workspace secrets to provision.
- App-only auth = no user context. All files in the tenant are accessible to the skill.

**When to use:** Any production enumeration, recursive sync, anything that needs to scale beyond a single folder.

### Common Token-Fetch Pattern (Azure AD)

```python
import requests

def get_access_token(tenant_id, client_id, client_secret):
    resp = requests.post(
        f"https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token",
        data={
            "grant_type": "client_credentials",
            "client_id": client_id,
            "client_secret": client_secret,
            "scope": "https://graph.microsoft.com/.default",
        },
    )
    resp.raise_for_status()
    return resp.json()["access_token"]
```

Token is short-lived (~1 hour). For long-running skills, refresh by re-calling.

---

## Microsoft Graph API Patterns

### Site / Drive Discovery

Composite site IDs are returned by a path lookup against the SharePoint hostname:

```
GET /v1.0/sites/{hostname}:{site_path}
# example: /v1.0/sites/contoso.sharepoint.com:/sites/Portal
```

Returns `{ "id": "hostname,guid1,guid2", "displayName": "...", "webUrl": "..." }`. The composite `id` (with commas) is used in subsequent calls.

Then list drives:
```
GET /v1.0/sites/{site_id}/drives
```

Returns one entry per document library. The `id` (starts with `b!`) is what you pass to drive-scoped endpoints.

### Listing Folder Children (with Pagination)

```python
url = (
    f"https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{folder_id}/children"
    f"?$top=500"
    f"&$select=id,name,size,createdDateTime,lastModifiedDateTime,folder,file,parentReference,webUrl"
)
items = []
while url:
    data = graph_get(url, auth_headers)
    items.extend(data.get("value", []))
    url = data.get("@odata.nextLink")
```

Notes:
- `$top=500` is the max page size Graph accepts.
- `$select` is a meaningful bandwidth optimization when enumerating thousands of items.
- For the drive root, use `/drives/{drive_id}/root/children`.
- `@odata.nextLink` is an absolute URL; do NOT re-apply your query string when paginating.

### Retry / Backoff for Throttling

Graph throttles aggressively at scale. Always honor `Retry-After` on 429s. Treat 503/504 as transient with exponential backoff.

```python
def graph_get(url, auth_headers, max_retries=5):
    for attempt in range(max_retries):
        resp = requests.get(url, headers=auth_headers)
        if resp.status_code == 429:
            retry_after = int(resp.headers.get("Retry-After", 2 ** attempt))
            time.sleep(retry_after)
            continue
        if resp.status_code in (503, 504):
            time.sleep(2 ** attempt)
            continue
        resp.raise_for_status()
        return resp.json()
    resp.raise_for_status()
```

### Folder Fingerprinting (Cheap Change Detection)

For top-level folder change detection, Microsoft Graph's `folder.childCount`, `lastModifiedDateTime`, and `size` form a reliable composite fingerprint without needing the delta query API. Comparing this against a cached state lets you skip re-pulling unchanged folders entirely -- saves ~99% of Graph calls after initial seed.

### Download URLs

SharePoint file metadata includes `@microsoft.graph.downloadUrl` -- a temporary, pre-authenticated CDN URL that bypasses auth. It expires after ~1 hour. Download immediately when the trigger fires; do not store for later use.

### Delta Query (Whole-Drive Change Tracking)

```
GET /sites/{siteId}/drive/root/delta
```

- Returns only items that changed since the last call.
- Deleted items are returned with a `deleted` facet.
- Follow `@odata.nextLink` until you receive `@odata.deltaLink`. Store the deltaLink for the next run.
- First run: pass `?token=latest` to skip the initial enumeration and only get changes from now on.

---

## Production Architecture Patterns

Three working patterns, in order of complexity:

### Pattern 1: SP File Listing (Single Agent, Recursive Enumeration)

A single agent with site/drive/folder IDs as code properties, a time trigger (hourly), and a recursive collection structure that descends up to 6 levels deep.

**When to use:** Small/medium drives (500 or fewer root folders), single source of truth for a finite directory, exploratory work.

**Limits:**
- Pipedream's 200-item cap forces Azure AD for larger folders
- 6-level nested-collection structure is awkward to query downstream
- Recursive collection fan-out fights V7's 1,000-entity collection cap once a drive grows past ~1,000 folders at any one level

### Pattern 2: SP Folder Store + SP Sync Orchestrator (Delta-Sync Cache)

The production architecture for large drives. Two agents:

**Agent A -- SP Folder Store (cache).** One entity per top-level folder. All properties `tool: manual`; written by the orchestrator's skills. Key fields:

| Property | Purpose |
|----------|---------|
| `folder_id` | SharePoint folder ID. Primary lookup key. |
| `folder_name` | Display name. |
| `file_tree` (json) | Full recursive nested-dict tree for this folder. |
| `file_count` / `folder_count` (number) | Recursive counts. |
| `last_successful_sync` (text) | ISO timestamp of last successful pull. |
| `last_error` (text) | Error message from the most recent failed sync. |
| `server_last_modified` / `server_child_count` / `server_size` | Server-side fingerprints for diff. |
| `sync_status` (single_select) | `pending` / `success` / `failed`. Self-healing signal. |
| `file_listing` (text) | YAML-formatted flat listing of all files with parsed fields. LLM-queryable. |
| `fund_identifiers` (json) | LLM-extracted identifiers for matching by fund name. |

**Agent B -- SP Sync Orchestrator (time-triggered).** A single all-in-one skill executes one orchestrator cycle:

1. Get Azure AD token.
2. List live root folders via Graph API (cheap metadata call).
3. Fetch the current cache from Agent A via V7 Go API.
4. **Diff** each live folder against its cached entry -- classify as `new`, `previous_failed` (self-healing retry), `changed`, or `unchanged` (skip).
5. **Sync** up to `max_per_run` folders. For each: recursively enumerate via Graph, count files/folders, write to Agent A via PATCH (existing) or POST (new). On exception, write `sync_status: "failed"` + `last_error` so the next cycle retries.
6. **Cleanup** entities for folders no longer in SharePoint.
7. Return a summary so the orchestrator can surface progress.

**Why this beats Pattern 1 at scale:**
- Unchanged folders are not re-pulled -- saves ~99% of Graph API calls after seed.
- Failed folders self-heal on the next tick.
- The 1,000-entity collection cap no longer binds.
- Per-folder errors are isolated.

### Pattern 3: Execute Skill API Bypass (Backfill / Unstuck)

When the LLM property tool wrapper refuses to execute (e.g., safety filter triggered by certain content patterns), bypass it by invoking the skill directly via the Execute Skill API:

```
POST /api/workspaces/{wid}/skills/{skill_id}/runs
{ "args": { "folder_id": "...", "folder_name": "...", "drive_id": "..." } }
```

This bypasses the LLM property-tool wrapper entirely. A driver script can use `ThreadPoolExecutor` to fan out folders with explicit concurrency control.

Use this pattern when:
- The LLM tool refuses to execute (safety filter or other intervention)
- You need to backfill a stuck cache quickly
- You want explicit concurrency control

### Bonus: Shallow Seed Pattern

Lists all root folders via one paginated Graph call and POSTs one shallow entity per folder to SP Folder Store (folder-id, folder-name, fingerprints, `sync_status: "pending"`) -- but skips the recursive `file_tree`. With `ThreadPoolExecutor(max_workers=10)` it can seed 2,000+ folders in under 90 seconds.

The next sync cycle classifies seeded entries as `previous_failed` and backfills the file trees over multiple cycles. Useful when downstream agents need folder names/IDs immediately.

---

## Downstream Integration Pattern

Example: an email triage agent consuming SP Folder Store to find files when a customer emails asking for a document.

1. **Document Search** (json) -- invokes skills to return all GPs/funds with entity IDs from SP Folder Store, then returns the YAML file listing for the matched entity.
2. **Document Match** (json) -- LLM matches the requested document against the YAML file listing. Returns `{found, file_id, file_name, match_reason}`.
3. **Document Status** (single_select) -- pulls Found/Not Found. Drives view scoping.
4. **File ID / File Name** (text) -- extracted from Document Match.
5. **Protected File** (file) -- downloads via Pipedream OAuth and password-protects.
6. The Send Document view emits the response email with the protected file attached.

This avoids full-tree fuzzy search by leveraging per-folder LLM-extracted `fund_identifiers` to find the right folder in one hop.

---

## SP Folder Store: `file_listing` (YAML Format)

A code property that turns the recursive `file_tree` JSON into a YAML-friendly flat listing, one line per file. Parsed fields (name, id, type, year, quarter, date) let downstream LLMs find files by date/quarter/type without re-parsing the tree.

```yaml
- name: Fund VII Q3 2024 Capital Account Statement.pdf
  id: 01ABCDEFG...
  type: pdf
  year: 2024
  quarter: Q3
  date: 2024-09-30
```

LLMs handle YAML markedly better than JSON for "find me the file matching X" queries because the flat indent-based structure is shorter and fields are aligned per line.

---

## Pipedream-Backed Skills

When using `Integration("sharepoint_online")`, you get a `PipedreamAuth` object with `session` and `auth_provision_id`. Two approaches:

**(a) Use `pipedream.session` directly** for raw Graph API calls -- works like the Azure AD bearer-token pattern but with Pipedream-provided OAuth credentials.

**(b) Wrap a Pipedream component** via `PipedreamStateManager` for managed action invocation. Key details:
- `Integration("sharepoint_online")` tells V7 Go to use the SharePoint Online Pipedream connector
- Dependencies: `["levenshtein", "pipedream"]`
- `allowed_domains: []` (Pipedream handles HTTP internally)
- `resolve_prop` does a name search. Skip it for raw IDs (composite, `b!`, or numeric)

### Binding Parameters with `argument_overrides`

When attaching a Pipedream-backed skill to a property, use `argument_overrides` to bind parameters directly rather than relying on the LLM to infer them:

```json
{
  "enabled_skills": [{
    "skill_id": "skill-uuid",
    "argument_overrides": [
      {"value": "user@example.com", "value_type": "fixed", "arg_type": "integration", "arg_name": "pipedream", "value_format": "default"},
      {"value": "@<P{site_id_prop}>", "value_type": "input_reference", "arg_type": "dynamic", "arg_name": "siteId", "value_format": "default"},
      {"value": "@<P{drive_id_prop}>", "value_type": "input_reference", "arg_type": "dynamic", "arg_name": "driveId", "value_format": "default"}
    ]
  }]
}
```

Bypasses LLM parameter mapping entirely. Recommended for any production wiring -- eliminates a class of hallucinated-argument bugs.

---

## Pipedream SharePoint Triggers

| Trigger | Component ID | Description |
|---------|-------------|-------------|
| New File Created | `sharepoint-new-file-created` | Fires when a new file is uploaded to a folder. |
| New Folder Created | `sharepoint-new-folder-created` | Fires when a new folder is created. |
| Updated File (Instant) | `sharepoint-updated-file-instant` | Fires on file update OR deletion. Requires pre-selecting specific `fileIds` at deployment time. |
| New List Item | `sharepoint-new-list-item` | Fires when a list item is created. |
| Updated List Item | `sharepoint-updated-list-item` | Fires when a list item is updated. |

**Important:** The `Updated File (Instant)` trigger's `fileIds` cannot be modified post-deployment. Workaround is delete + redeploy.

---

## Key Gotchas

### Pipedream / Auth

**0. Always pass raw Graph API IDs as `PipedreamRawValue`.** This is the most common source of breakage. Pipedream's `resolve_prop` searches by name, which fails for raw IDs. Detection patterns:
- Site IDs contain commas (`hostname,guid,guid`) --> `PipedreamRawValue`
- Drive IDs start with `b!` --> `PipedreamRawValue`
- Folder/item IDs are always opaque --> always `PipedreamRawValue`, never `resolve_prop`

Wrap as `{"__type": "PipedreamRawValue", "value": id_string}` via `state_manager.add_prop()` directly.

**1. Pipedream silently caps at ~200 items per call.** Setting `$top` higher has no effect. Move to Azure AD client credentials if you need more.

**2. Pipedream trigger props are immutable after deployment.** Cannot add new `fileIds` to triggers. Delete and redeploy is the only path.

### Microsoft Graph

**3. `$top=500` is the hard maximum page size.** Setting higher just clamps to 500.

**4. `@odata.nextLink` is absolute.** Don't re-apply your query string.

**5. Throttle aggressively (429/503/504).** Always honor `Retry-After` and back off exponentially.

**6. `@microsoft.graph.downloadUrl` expires after ~1 hour.** Download immediately.

**7. Composite site IDs and `b!` drive IDs are not searchable.** Use path-based discovery to obtain them, then hard-code.

### V7 Go Platform

**8. The 1,000-entity collection cap binds at scale.** Use a flat top-level agent (like SP Folder Store) rather than nested collection fan-out.

**9. The 60-second gateway timeout on skill invocations.** Big recursive enumerations time out at the gateway even though the skill keeps running server-side. Split into per-folder skills.

**10. LLM property tools can refuse skill execution.** Observed when result payloads contain certain content patterns. Mitigation: bypass the LLM wrapper by invoking via `POST /skills/{id}/runs`.

**11. LLM-mediated tools can invoke a skill multiple times within a single property execution.** Skills that write to external state MUST be idempotent. Defensive patterns:
- Fetch cache at the top of the skill
- After every successful POST, add to in-memory cache
- Before any POST, do a cheap GET filter by lookup key

**12. Hub PUT only renames.** To update file content, delete the old file and re-upload. Reindex after.

**13. `allowed_domains` for CDN downloads.** Skills downloading from `@microsoft.graph.downloadUrl` need the CDN host whitelisted. The CDN domain varies. Easiest: use `["graph.microsoft.com", "login.microsoftonline.com"]` for Azure AD skills plus the CDN host.

**14. `tool_metadata.skill_results` for reading skill results in code properties.** Code properties downstream of a skill should reference results as:
```python
data = @<P{prop_id}:Ttool_metadata.skill_results>[0]["raw_response"]
items = data.get("result", [])
```
Without this, code properties only see the LLM's natural-language summary and lose structured data.

---

## Microsoft Graph API Quick Reference

### Key Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /sites/{hostname}:{path}` | Look up site ID from hostname + relative path |
| `GET /sites/{siteId}/drives` | List drives (document libraries) in a site |
| `GET /drives/{driveId}/root/children` | List immediate children of the drive root |
| `GET /drives/{driveId}/items/{itemId}/children` | List immediate children of a specific folder |
| `GET /drives/{driveId}/items/{itemId}` | Get a single drive item by ID |
| `GET /sites/{siteId}/drive/root/delta` | Delta query for tracking all file changes |
| `POST .../oauth2/v2.0/token` | Get a bearer token (client credentials grant) |

### Query Parameters

- `$top=500` -- page size (max)
- `$select=id,name,size,...` -- narrow projection
- `$orderby=lastModifiedDateTime desc` -- sort
- `$filter=...` -- server-side filtering

---

## Future / Sketched Patterns

- **Hub-per-folder ingestion** -- A New-Folder-Created trigger creates a V7 hub per SharePoint folder and provisions a child agent for file ingestion. Worked for small-scale tests but never used at production scale.
- **Full delta-sync via Graph delta query** -- Per-file change detection across deep trees. Would require persisting the `@odata.deltaLink` between runs.
