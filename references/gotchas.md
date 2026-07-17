# V7 Go Gotchas Reference (G1-G74)

A searchable catalog of every known V7 Go platform quirk, trap, and hard-won lesson. These are numbered G1 through G74 so you can reference them quickly (e.g., "watch out for G55" in a prompt or plan).

**How to use this file:** Scan the Quick Reference for the most common issues, then browse by category when you hit something specific. Each gotcha includes a plain-language summary so you know what can go wrong even if the technical details don't apply to you.

---

## Quick Reference: Top 10 Gotchas

These are the ones that trip people up most often. If you read nothing else, read these.

| # | Summary | Why It Matters |
|---|---------|----------------|
| **G1** | Always verify which workspace you're in before making changes | Creating things in the wrong workspace is a painful, manual cleanup |
| **G5** | Every property needs a `description` field | Without it, the API rejects the request entirely |
| **G6** | The type name is `single_select`, not `select` | Using the wrong name causes a cryptic error |
| **G8** | Updates require the full property definition, not just changed fields | Partial updates silently fail or break things |
| **G42** | Never use the clone API for copying workflows | It produces broken property chains and orphaned references |
| **G55** | Never use `auto_llm` (the generic "AI" tool) | It picks a random model each time, making output unpredictable and impossible to debug |
| **G72** | Always set `default_view_id` when creating properties | Without it, properties land in the main view and clutter the user's workspace |
| **G73** | Properties only run when the entity is routed to their view | If the view filter doesn't match, the property silently does nothing |
| **G16** | Always wrap skill code in error handling | Without it, errors produce generic messages that are impossible to debug |
| **G53** | Pagination uses `offset`, not `start_cursor` | The API response includes `start_cursor` but the request rejects it |

---

## Workflows & General Safety

**G1. Always verify which workspace you're targeting before making any changes.**
When organizations have multiple workspaces (common in enterprise setups), the names can look almost identical. Before you create or modify anything, list your workspaces, match by name, and confirm both the name and ID. Creating resources in the wrong workspace violates data isolation and requires manual cleanup.

**G2. Save everything locally before pushing to V7 Go.**
Keep a local copy of every property, skill, and configuration in your project folder. This is your safety net and audit trail. Never push changes to V7 Go without saving them locally first.

**G3. Always check the current state before updating anything.**
Before updating any resource on V7 Go, fetch the latest version first. Someone may have made changes through the UI that you don't know about. Overwriting those changes silently is a common source of frustration.

**G4. Read-only workspaces are off-limits for changes.**
If a workspace is designated as read-only (e.g., a reference or production workspace), only read from it. Never create, update, or delete anything there, even if the API would technically allow it.

---

## Properties

**G5. Every property must include a `description` field when you create it.**
Even for simple text properties, the API requires a `description`. Omit it and you get a "Missing field: description" error. This is a required field, not optional.

**G6. The correct type name is `single_select`, not `select`.**
A common mistake: using `"select"` instead of `"single_select"` in the API. The wrong name produces a confusing "union cast error." Similarly, use `"multi_select"` (with the underscore), not `"multiselect"`.

**G7. Properties without inputs won't run automatically.**
A property only executes when its input properties finish computing. If you want a property to be part of your workflow pipeline, it needs at least one entry in its `inputs` array. Otherwise it just sits there doing nothing.

**G8. Property updates require the complete definition, not just the fields you changed.**
When updating a property, you must send the entire property definition (name, type, tool, description, inputs, enabled_skills, config). Sending only the changed fields causes missing-field errors. Think of it as "replace the whole thing" rather than "edit one part."

**G9. Avoid the `integration` property type.**
It's unstable. Use `json` or `text` properties with a `code` tool instead, and access data through property references (`@<P...>`). This gives you the same functionality with much more reliability.

**G10. File-output skills can only attach to `file`-type properties.**
If a skill produces a file as output (using `output_type: "file"`), it can only be used on a `file`-type property. If you need file download functionality in a `json` property, build a custom skill that handles the download internally and returns JSON instead.

**G59. When updating properties, pass `config: {}` unless you're specifically changing config.**
The update endpoint rejects certain config fields that are only valid at creation time. For select properties, it rejects `config.options` and `config.default_option`. For collections, it rejects `config.properties` and `config.subproject_config`. Passing an empty `config: {}` tells the API to keep the existing configuration unchanged. To modify select options on an existing property, use `config.upsert_options` (to add/update) and `config.remove_options` (to delete) instead of `config.options`.

**G60. Renaming a property changes its slug (the URL-friendly identifier).**
The property ID stays the same, so references like `@<P{id}>` and view filters keep working. But anything that uses the old slug (skill code, field value lookups, view layout scripts) will break. Before renaming: search your workspace for the old slug, update every reference, republish affected skills, then force-recalculate.

---

## Skills & Custom Code

**G11. Use `argument_overrides` to pass known values to skills, not LLM inference.**
When a skill's parameters are known ahead of time (hardcoded values, property references, or secrets), bind them directly using `argument_overrides`. This bypasses the LLM entirely for parameter mapping — the LLM only decides whether to run the skill, not what values to pass. Only fall back to LLM-inferred parameters when the values are truly dynamic and can't be determined at design time.

Override types:
- `"arg_type": "integration"` + `"value_type": "fixed"` — OAuth account (value = connected account email)
- `"arg_type": "dynamic"` + `"value_type": "input_reference"` — property reference (value = `@<P{id}>`)
- `"arg_type": "dynamic"` + `"value_type": "fixed"` — hardcoded string value
- `"arg_type": "secret"` + `"value_type": "fixed"` — workspace secret (value = secret name)

**G12. List every external domain your skill contacts in `allowed_domains`.**
If your skill makes HTTP requests to any external service, those domains must be in the `allowed_domains` list. The domain `go.v7labs.com` covers V7 Go API calls. Missing a domain means the request silently fails or returns an HTML error page.

**G13. `V7Go.metadata` and `Secret()` only exist at runtime.**
These are injected by V7 Go's execution environment. They don't exist when you're testing locally. Write skills that can accept these as parameters for local testing.

**G14. Accessing metadata requires `.__dict__`, and `api_base_url` is just the domain.**
The metadata object doesn't support bracket notation — `V7Go.metadata["workspace_id"]` throws an error even though the docs suggest otherwise. Use `V7Go.metadata.__dict__["workspace_id"]` instead. Also, `api_base_url` returns just `https://go.v7labs.com`, not the full API path. You need to build the full URL yourself:
```python
base = V7Go.metadata.__dict__["api_base_url"]
wid  = V7Go.metadata.__dict__["workspace_id"]
api_url = f"{base}/api/workspaces/{wid}"
```
Using `base` directly (without `/api/workspaces/...`) hits the frontend HTML and produces a JSON parse error.

**G15. Be careful with string escaping in skill code.**
When skill code is embedded inside a JSON payload, string escaping becomes multi-layered. Use raw strings or write code to files to avoid shell interpolation issues.

**G16. Always wrap skill logic in try/except blocks with structured error returns.**
The V7 Go skill sandbox does not surface tracebacks. If your code throws an unhandled exception, you get a generic "skill execution error" with no details. Every skill should catch exceptions at each logical step and return a structured error dictionary including the error message, status code, and traceback.

**G17. Skills can't write to the filesystem.**
Custom skills run in a sandboxed environment with no filesystem access. You can't create temporary files, use `tempfile`, or write to `/tmp`. All file generation must happen entirely in memory using `BytesIO`:
```python
from io import BytesIO
buf = BytesIO()
# ... generate content into buf ...
return V7Go.output_file(filename="report.docx", bytes=buf.getvalue())
```

**G18. Store binary assets (like logos) as base64-encoded secrets.**
If a skill needs a binary file (like an image), encode it as base64, store it as a workspace secret, and decode it at runtime. Don't rely on co-located files — the sandbox doesn't guarantee they'll be available.

**G56. Prefer `V7Go.call_skill` for deterministic skill execution.**
Calling a skill from a `code` property using `V7Go.call_skill(SKILL_ID, **kwargs)` removes the LLM from the process entirely — it's direct, predictable, and faster. The skill still needs an `enabled_skills` entry (for authorization and secret injection), but the LLM never touches the dispatch. Use LLM-hosted skill invocation (G11) only when the skill needs natural-language judgment to decide whether to fire, or for file-output skills (which require an LLM host).

**G57. How you read a skill's output depends on how it was called.**
If the skill was called via `V7Go.call_skill` (deterministic), read its output with the plain `@<P{id}>` value tag — the field value IS the skill's return envelope. If the skill was called via LLM-hosted invocation, you must read it through the legacy path (`:Ttool_metadata.skill_results`). Mixing these up returns empty results every time.

**G58. File-type fields use signed S3 URLs, not direct downloads.**
When a skill reads a `data`/`file` field, the value is a pre-signed S3 URL stored in `manual_value.value` (a string). Fetch this URL WITHOUT the V7 Go API key (the URL carries its own auth). The filename isn't at the top level — extract it from the URL's `response-content-disposition` parameter. Make sure to include the storage host (e.g., `s3.eu-west-1.amazonaws.com`) in `allowed_domains`, or the download returns an HTML error page instead of the file.

---

## Views & Conditional Workflows

**G31. View filters use a different format than entity list filters.**
For filtering views by select values, use `{"property_id": "...", "select_option_value": "Option Name"}`. Don't use the nested `subject`/`matcher` structure that the entity listing endpoint expects — they're different formats for different endpoints.

**G32. View updates always require `property_ids` and `property_layouts`.**
Even when you're only changing filters, the PUT endpoint requires both `property_ids` (array of UUIDs for visible properties) and `property_layouts` (array, can be empty `[]`) alongside `filters`. Omitting them causes the update to fail.

**G33. Multiple view filters are combined with AND logic.**
If you put multiple filter objects in a view's `filters` array, they're ANDed together. An entity must match ALL filters to appear in the view.

**G34. Select option colors use `rainbow-XX` format, not color names.**
Don't use plain color names like `"green"` or `"blue"`. The API expects `rainbow-01` through `rainbow-17`. When updating existing options, use `config.upsert_options` (not `config.options`).

**G35. Conditional workflows are built using views.**
Each select option value can map to a view with its own set of visible properties. This is the core mechanism for building branching logic — different document types, different processing paths, different review stages.

**G36. Properties outside an entity's routed view are "skipped."**
When an entity doesn't match a particular view's filter, all properties scoped to that view are marked as "skipped" — they never calculate, and anything downstream that depends on them also won't run. This is expected behavior, not a bug.

**G37. Use a code property on the main view to aggregate results from branching views.**
To bring results from different branches back together, create a `code` property that lists all branch-specific properties as inputs and picks whichever is non-null. This property must be on a view with no filter (typically the main view) so it runs for all entities:
```python
result = "No result"
for val in [@<P{branch_a_id}>, @<P{branch_b_id}>, @<P{branch_c_id}>]:
    if val:
        result = val
```

**G38. Select values in code properties are lists, not strings.**
When you read a `single_select` or `multi_select` property in a `code` property, the value comes back as a list (e.g., `["STP"]`), not a bare string. Always compare with list form: `if route == ["STP"]`, not `if route == "STP"`.

**G72. Always set `default_view_id` when creating properties that don't belong in the main view.**
If you create a property without specifying `default_view_id`, it automatically lands in the main view. This clutters the main view with intermediate/processing properties that users shouldn't see. Pass `"default_view_id": "<target_view_id>"` in the create request to place the property in the correct processing view instead.

If a property has already leaked into the main view, you need to explicitly remove it — adding it to another view is NOT enough, it stays in main until you remove it. After creating or moving any property, re-read the main view and confirm only the intended top-level properties are visible. When in doubt about whether something belongs in main, it doesn't — keep main minimal.

**G73. Properties only compute for entities that are routed to their view.**
This is a critical concept. The main view is always active for every entity. Every other view is a "routed" view: it must filter on a routing property (usually a `single_select`), and a property in that view only computes when the entity's routing value matches the filter.

Key mechanics:
- A non-main view with empty filters (`filters: []`) computes for NO entities — every recalculation returns `inactive_view`.
- To make a processing view active: set its filter to a routing property + option value. Example: an "Excel Processing" view filters on `Input File Type = "excel"`, so only Excel files trigger those properties.
- If you delete and recreate a routing property (which gives it a new ID), every view that filtered on the old ID silently reverts to empty filters and stops working. After recreating a router, re-set every dependent view's filter to the new ID.
- Routing controls activation; stage-gate dependencies control ordering within a view. They're independent mechanisms.
- Always check `ignored_reasons` for `inactive_view` after recalculating to verify routing is working correctly.

---

## Triggers

**G19. Inbound triggers create a "backing project" but don't auto-create the trigger property via API.**
In the UI, the `trigger` property (type: trigger, owner: system) is auto-created. Via the API, it isn't. Instead, create a `reference` property pointing to the backing project and use it as `via_property_id` for inputs that need to reach the webhook data.

**G20. Trigger data lives in the backing project, not the main project.**
The actual webhook data is stored in the backing project. To find it, enable the trigger — the response includes `backing_project_id`. Then list that project's properties to find the `Webhook Data` property.

**G21. Triggers must be explicitly deployed after setup.**
Creating and configuring a trigger isn't enough. You must explicitly deploy it afterward. The deploy call specifies `target_action: "create_entity"`.

---

## Collections

**G24. Use the child project ID when working with collection entities.**
When working with entities inside a collection, use the child project's ID (found at `config.subproject_config.child_project_id`), not the parent project's ID. Using the parent ID targets the wrong set of entities.

**G25. Collection entities require a `parent_entity_id`.**
When creating entities in a collection sub-project, you must provide `parent_entity_id` pointing to the parent entity. Without it, the entity has no parent and won't be associated correctly.

**G67. Collection child properties created by LLM output can't be retyped in place.**
When an LLM collection declares its schema via `config.properties`, V7 creates system-owned children that can't have their type changed via the API. Workaround: add a sibling `code` property in the same child project that reads the source via a property reference and transforms the data as needed.

**G68. Code-tool collections need their columns defined at creation time, and each branch needs its own view.**
A code-tool collection maps its output to child columns BY NAME — but only for columns listed in `config.properties` at creation time. Columns added later aren't registered, so rows stay idle forever (you can't fix this with an update — you must recreate the collection). Also, fan-out/fan-in branches deadlock if every property sits only in main; give each branch its own filtered view so unused branches resolve to "skip" and the pipeline proceeds.

**G74. For mixed-file collections, use `never` (not `drop_any_incomplete`) on cross-row fan-ins.**
When a pipeline processes a mix of file types (Excel + PDF + images + email), some rows won't apply to certain branches and their stage properties never complete. A `drop_any_incomplete` fan-in fires as soon as the first row completes and produces a partial result. A `never` fan-in excludes rows that are `inactive_view` (routed out of the property's view), so it correctly waits only for applicable rows. Use `never` for stage-gate aggregators in mixed-collection pipelines.

---

## Hubs (Knowledge Libraries)

**G26. Hub file updates only support renaming.**
The update endpoint for hub files only changes the file's name. To update actual file content, delete the old file and upload the new version.

**G27. Reindex hubs after adding or removing files.**
After any changes to a hub's file list, call the reindex endpoint to update the search index. Without this, the hub's AI search won't reflect your changes.

**G28. Hubs have a 1,000-item limit.**
Each hub can hold a maximum of 1,000 files. If you're approaching this limit, check the count before uploading more files.

---

## API Responses & Entity Data

**G22. Property list responses have inconsistent formats.**
The response from listing properties may be either a direct array or an object with a `data` key. Always handle both:
```python
props = response if isinstance(response, list) else response.get("data", [])
```

**G23. Retries can create duplicate properties.**
If API calls are retried due to timeouts, you can accidentally create the same property twice. Always check for existing properties by slug before creating new ones.

**G45. Entity updates use PUT, not PATCH.**
To update an existing entity, use `PUT /entities/{eid}`. PATCH returns 404. Both create and update take `{"fields": {...}}` keyed by property slug.

**G46. Entity field value formats depend on the property type.**
Different property types require different value formats:
- **text**: bare string — `{"my-text": "hello"}`
- **number**: bare integer — `{"my-num": 42}`
- **single_select / multi_select**: array of strings — `{"my-select": ["success"]}` (not a bare string)
- **json**: object with a `json` key containing stringified JSON — `{"my-json": {"json": "{\"a\":1}"}}`

Wrapping any of these in `{"value": ...}` is rejected.

**G53. Pagination uses `offset`, not the `start_cursor` from the response.**
The entity list API response includes `start_cursor` and `end_cursor`, but the request side rejects `start_cursor` as an unexpected field. Use offset-based pagination instead: `params={"limit": 750, "offset": offset}`. Check `body.get("metadata", {}).get("has_next_page")` to know when to stop.

**G54. Smaller page sizes are often faster.**
Benchmarked against ~1,000 entities: `limit=250` is fastest (~4 seconds), `limit=750` is a good default (~6.4 seconds), and `limit=999` is slowest and risks errors. Use `limit=750` as your default.

**G69. Setting a reference field manually requires a specific schema.**
You can't just pass a list of IDs or use `{matched_entity_ids}`. Reference fields require the `TYPED_REFERENCE` format with filters:
```json
{"fields": {"<ref-slug>": {"reference": {"filters": [{"subject": "entity_id", "matcher": {"name": "any_of", "values": ["<eid>"]}}], "conjunction": "and"}}}}
```

---

## Testing & Recalculation

**G39. Use per-entity recalculation for testing, not recalculate-all.**
Target specific entities and properties for focused testing:
```
POST /projects/{project_id}/entities/{entity_id}/recalculate
{"property_ids": ["prop-id-1"], "force": true}
```
The `force: true` flag recalculates even if the field isn't marked as stale.

**G40. For collections, recalculate child entities individually.**
List the child entities for the specific parent, then recalculate each one. Don't try to recalculate the parent and expect children to update automatically.

**G41. Wait for computations to finish before triggering downstream properties.**
Check the entity field's `status` — wait until it shows `complete` (not `computing` or `idle`) before recalculating downstream properties. Otherwise you'll get stale or empty results.

**G61. Code-only edits don't trigger recomputation.**
A `code` property re-runs only when its INPUT SET changes. If you change the code but not the inputs, the old cached result is served even with `force: true`. To bust the cache, add or remove a throwaway input. The cleanest alternative when iterating heavily: delete and recreate the property (but this breaks view columns and downstream references).

**G62. `skip_behaviour` only responds to `tool_fallback: true` signals from single_select properties.**
The skip mechanism only works with a specific signal: a `single_select` property whose selected option has `tool_fallback: true`. A text or code property emitting `"-"` doesn't trigger skip behavior. The only way to fully skip a property for certain entities is view routing — place the property only in a filtered view, and entities outside that view never schedule it.

**G63. Choose `never` vs `drop_any_incomplete` carefully for collection inputs.**
- `never`: Wait for ALL child rows before running. Use this when a downstream property must aggregate every row.
- `drop_any_incomplete`: Run immediately with whatever rows are complete. Use this ONLY when upstream branches are mutually exclusive (one row fires, the rest legitimately don't apply).
Getting this wrong silently corrupts aggregations — the property fires with partial data and doesn't recompute when the rest arrives.

**G64. Editing a property's code may not invalidate existing results.**
After changing a property's logic, existing per-entity values may not update automatically. `recalculate` with `force: true` can return `affected_count: 0`. For collections, updating the collection code doesn't mark existing child rows as stale — you may need to delete child entities and recalculate. New submissions process correctly; old rows often need manual intervention.

**G65. New LLM properties won't auto-compute for existing entities.**
Code properties tend to compute on read, but LLM-tool properties stay empty for existing entities. You need to force-recalculate with both `property_ids` and `force: true` to populate them.

**G66. Large extractions can hang in a "zombie" computing state.**
If a grounded LLM extraction is processing a very large set of chunks, it can get stuck in `computing` status indefinitely (no error, never resolves). Detect this by checking if `status` is `computing` but `updated_at` hasn't advanced for several minutes while all upstream inputs are complete. Fix by force-recalculating the property. Mitigation: avoid feeding 50+ chunks to a single extraction — use hierarchical reduce instead.

---

## Cross-Workspace & Migration

**G42. Never use the clone API for cross-workspace migration.**
The `POST /projects/{id}/clone` endpoint is unreliable — it produces broken property chains, orphaned trigger references, and stale skill IDs. It only works within the same workspace anyway. Always rebuild property-by-property instead.

**G43. Property IDs are workspace-scoped and don't transfer.**
An ID from one workspace is meaningless in another. All property references (`@<P{id}>`) and hub references (`@<H{id}>`) must be translated to the target workspace's IDs after recreation.

**G44. Secrets must be created separately in each workspace.**
A secret in one workspace is not available in another. Create secrets using `PUT /api/workspaces/{wid}/secrets/{secret_name}` (the name goes in the URL path, not the body). The POST endpoint doesn't work (returns 404).

**G71. V7 Go is regionally isolated — EU and US are completely separate.**
`go.v7labs.com` (EU) and `go.us.v7labs.com` (US) are separate deployments with separate data stores and separate API keys. An EU key against the US host returns 401 and vice versa. Everything is region-scoped: secrets, hub content, all resource IDs must be recreated in the target region. Skills are portable across regions only if they never hardcode the domain — use `V7Go.metadata.__dict__["api_base_url"]` at runtime.

---

## Concierge / Email to Chats

**G50. The file property must be named exactly "File" for Concierge ingestion.**
When using the email-to-chats feature (Concierge), the `file`-type property that receives email attachments must be named exactly `File`. Concierge uses this name to deposit the selected attachment.

**G51. Write a specific agent description — Concierge uses it to pick the right attachment.**
Concierge reads the agent's description to decide which attachment from an email is the correct one to process. A vague description means wrong file selection. Write a specific, actionable description.

**G52. The email sender must have a V7 Go account.**
Emails from addresses not associated with a V7 Go account in the workspace won't be processed. This is a common source of "nothing happened" issues.

---

## LLM Behavior & Model Selection

**G47. Skill arguments get truncated for large list inputs.**
When an LLM property passes a large list to a skill via `argument_overrides`, the list gets truncated before delivery (observed: 2,060 items truncated to 16). Workarounds: combine producer and consumer into one skill, cap the producer's output to under 100 items, or persist intermediate state via API writes.

**G48. LLM properties can invoke a skill multiple times in a single execution.**
If a skill creates entities via the API, the same record can be written multiple times, producing duplicates. Make your writes idempotent: look up by a stable business key before any POST, convert to PUT if the entity exists, and update your in-memory cache after every POST.

**G49. LLM properties can refuse to execute a skill for "safety" reasons.**
Some model providers (observed with GPT models) refuse to run a skill if the description or argument names trip a safety filter. The skill just doesn't execute with a `request_refused` error code. Workaround: use the Execute Skill API directly to bypass the LLM entirely.

**G55. Never use `auto_llm` (the "AI" tool shown in the UI).**
This setting auto-selects a model at runtime, producing non-deterministic output that is impossible to debug or reproduce. Always specify a model by name (e.g., `gpt_5_mini`, `gemini_2_5_pro`, `claude_4_sonnet`). For simple extraction of a single value from JSON, use `text/code` instead — it's cheaper, faster, and completely deterministic.

---

## Project Naming & Settings

**G29. Keep project names short.**
Agent/project names should be a maximum of 3 words. Descriptions should be a maximum of 2 sentences. Short names display better in the UI and are easier to reference.

**G30. Don't modify advanced project settings.**
Fields like `auto_recalculations`, `published`, and other advanced project settings should never be changed unless the user explicitly requests it. Changing these can have unexpected side effects on workflow behavior.

---

## Platform Traps

**G70. Two infrastructure-level traps to watch for.**

**(a) AWS WAF blocks certain words in API request bodies.**
The platform's web application firewall (AWS WAF) treats quoted SQL reserved words in request bodies as potential attacks. If your property code contains a literal string like `'limit'`, `'select'`, `'order'`, or `'where'`, the request is rejected with a generic `403` error. The error message is an HTML page, not a helpful API error. Workaround: build the problematic string from split tokens (e.g., `k = 'li' + 'mit'`) and avoid using SQL keywords as property slugs or names.

**Plain-language summary:** The platform's security layer is overly aggressive about SQL-like words. If you get a mysterious 403 error when saving code that contains words like "limit" or "select," this is probably why. Split the word into pieces in your code to work around it.

**(b) Entity IDs encode creation time, and empty values cause problems.**
Entity records don't have a `created_at` field, but entity IDs are time-ordered (UUIDv7), so you can sort by ID to recover creation order. Also, text and JSON field values can't be empty string or null — an empty code result stalls downstream properties. Use `"-"` or `"UNKNOWN"` as a no-value sentinel and treat it as empty when forwarding data.
