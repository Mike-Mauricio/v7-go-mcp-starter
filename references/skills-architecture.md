# Skills Architecture Reference

Skills are reusable capabilities that extend what V7 Go agents and chats can do. They are custom Python functions that run in a secure, sandboxed environment inside V7 Go. When your workflow needs to call an external API, transform data in a specific way, generate a file, or do anything beyond what the built-in AI models handle, you build a skill.

Think of skills as plug-in tools. You write the logic once, and any agent or property in your workspace can use it.

**Documentation:** [Skills](https://docs.go.v7labs.com/docs/skills-1)

---

## Skill Structure

A skill is a Python function named `run` that accepts typed parameters and returns a result. Here is the basic shape:

```python
from typing import Annotated

def run(
    document_text: Annotated[str, "The text to process"],
    api_key: Annotated[str, Secret("MY_API_KEY")]
):
    # Your logic here
    return {"status": "success", "result": "some value"}
```

Every skill has:
- **Parameters** -- typed inputs with descriptions. The `Annotated` syntax tells V7 Go what each parameter is for.
- **Return value** -- a JSON-serializable dictionary (or a file output, covered below).
- **Dependencies** -- any Python packages the skill needs (e.g., `["requests", "openpyxl"]`).
- **Allowed domains** -- which external services the skill is permitted to contact.

---

## Core Skill Features

### V7Go.metadata -- Accessing Workspace Context

Skills can access information about the workspace and agent they are running inside. The `V7Go.metadata` object provides:

- `api_base_url` -- the V7 Go instance URL (e.g., `https://go.v7labs.com` or `https://go.us.v7labs.com`)
- `workspace_id` -- the current workspace UUID
- `project_id` -- the agent/project the skill is running in
- `entity_id` -- the entity (row) being processed
- `property_id` -- the property that invoked the skill

**Critical: How to access metadata values**

The `V7Go.metadata` object is **not** a dictionary. You cannot use bracket notation like `V7Go.metadata["workspace_id"]` -- this will raise a `TypeError`. Instead, always access values through `.__dict__`:

```python
base = V7Go.metadata.__dict__["api_base_url"]
wid = V7Go.metadata.__dict__["workspace_id"]
```

Also note that `api_base_url` returns just the domain (e.g., `https://go.v7labs.com`), not the full API path. You need to build the full URL yourself:

```python
base = V7Go.metadata.__dict__["api_base_url"]
wid = V7Go.metadata.__dict__["workspace_id"]
api_url = f"{base}/api/workspaces/{wid}"
```

### Secret() -- Secure Credential Access

Skills access sensitive values (API keys, tokens, passwords) through workspace secrets. Instead of hardcoding credentials, you declare them as secret parameters:

```python
def run(
    api_key: Annotated[str, Secret("GO_API_KEY")],
    oauth_token: Annotated[str, Secret("MS_GRAPH_TOKEN")]
):
    # api_key and oauth_token are injected securely at runtime
```

Secrets are stored in the workspace and injected by V7 Go when the skill runs. They are never exposed in logs, and their values cannot be read back after creation.

**Important:** Both secrets and skills have their own `allowed_domains` settings. A secret's `allowed_domains` controls which hosts the secret value can be sent to. A skill's `allowed_domains` controls which hosts the skill's code can contact. Both must be set correctly.

### allowed_domains -- External API Access

Every skill that makes HTTP requests to external services must declare which domains it will contact. This is a security control -- the sandbox blocks requests to domains not on the list.

```json
{
  "allowed_domains": ["go.v7labs.com", "graph.microsoft.com", "api.openai.com"]
}
```

Common domain requirements:
- `go.v7labs.com` or `go.us.v7labs.com` -- for skills that call the V7 Go API
- `s3.eu-west-1.amazonaws.com` (or equivalent) -- for downloading files from V7 Go signed URLs
- `graph.microsoft.com` + `login.microsoftonline.com` -- for Microsoft integrations

**Never leave `allowed_domains` empty or null on a production skill.** An empty list means the skill could potentially contact any host, which is a security risk.

### V7Go.output_file() -- Returning Files

When a skill needs to produce a file (a generated report, a converted document, an exported spreadsheet), use `V7Go.output_file()`:

```python
from io import BytesIO

def run(...) -> V7Go.OutputFile:
    # Generate your file in memory
    buf = BytesIO()
    # ... write content to buf ...
    return V7Go.output_file(filename="report.docx", bytes=buf.getvalue())
```

Key rules for file-output skills:
- The return type annotation `-> V7Go.OutputFile` is required. It tells V7 Go this skill produces a file.
- File-output skills must be hosted on a `file`-type property with an LLM tool (e.g., `file/gpt_5_mini`). A `code`-tool property will not dispatch a file-output skill.
- All file generation must happen **in memory** using `BytesIO`. The sandbox does not allow filesystem access -- no `tempfile`, no writing to `/tmp`.

---

## Two Ways to Invoke Skills

### Option 1: V7Go.call_skill (Deterministic -- Preferred)

`V7Go.call_skill()` invokes a skill directly from a `code`-tool property with **no AI model in the loop**. This is the preferred approach whenever you know exactly what arguments to pass at design time.

```python
result = V7Go.call_skill('skill-uuid-here',
    param1="value1",
    param2=str(@<P{some_property_id}>).strip(),
)
```

**Why this is preferred:**
- No token cost (no LLM involved)
- No risk of the AI hallucinating parameter values
- The field value IS the skill's return value -- clean and direct
- Deterministic behavior every time

**Requirements:**
- The skill ID must be a **hardcoded string literal** in the code. The runtime rejects variables.
- The `enabled_skills` array on the hosting property must still include the skill (for authorization and secret injection), even though the LLM is not doing the dispatching.
- Non-secret arguments are passed as keyword arguments in the `call_skill()` call.
- Secret arguments are bound through `argument_overrides` in `enabled_skills` with `arg_type: "secret"`.

**Reading the result downstream:** Use the plain value reference `@<P{caller_property_id}>` and index into the returned dictionary. Do not use the legacy `tool_metadata.skill_results` path -- that only works for LLM-hosted skills.

### Option 2: LLM-Hosted (enabled_skills + argument_overrides)

When a skill needs to be invoked based on the AI model's judgment (not predetermined logic), you host it on an LLM-tool property. The model decides when to call the skill and can infer some parameter values from context.

**Best practice:** Even with LLM hosting, use `argument_overrides` to bind as many parameters as possible directly. This minimizes what the LLM needs to figure out:

```json
{
  "enabled_skills": [{
    "skill_id": "skill-uuid",
    "argument_overrides": [
      {
        "arg_name": "api_key",
        "arg_type": "secret",
        "value": "GO_API_KEY",
        "value_type": "fixed",
        "value_format": "default"
      },
      {
        "arg_name": "document_id",
        "arg_type": "dynamic",
        "value": "@<P{property_id}>",
        "value_type": "input_reference",
        "value_format": "default"
      }
    ]
  }]
}
```

**Override types at a glance:**

| arg_type | value_type | Use case |
|----------|------------|----------|
| `secret` | `fixed` | Workspace secret (value = secret name) |
| `dynamic` | `input_reference` | Property reference (value = `@<P{id}>`) |
| `dynamic` | `fixed` | Hardcoded string value |
| `integration` | `fixed` | Pipedream OAuth account (value = connected account email) |

---

## Skill Gotchas

These are critical lessons from real V7 Go deployments. Review them before building or debugging skills.

### G11: Use argument_overrides, Not LLM Inference

When a skill's parameters are known at design time (hardcoded values, property references, or secrets), always bind them via `argument_overrides`. This bypasses the LLM for parameter mapping entirely. Only fall back to letting the LLM infer parameters when they truly cannot be determined at design time.

### G12: Declare All External Domains

If your skill makes HTTP requests to any external service, every domain must be in `allowed_domains`. Missing a domain will cause silent failures. The `go.v7labs.com` domain covers V7 Go API calls; storage domains (like S3) are needed for file downloads.

### G13: V7Go.metadata and Secret() Are Runtime-Only

These objects are injected by the V7 Go execution environment. They do not exist during local development or testing. Write skills that accept these as parameters so you can test locally by passing values directly.

### G14: V7Go.metadata Access Pattern

The `Metadata` object is NOT subscriptable. `V7Go.metadata["workspace_id"]` raises a `TypeError`. Always use `V7Go.metadata.__dict__["key"]`. Also, `api_base_url` returns just the domain -- build the full API path yourself.

### G15: String Escaping in Skill Code

When skill code is embedded in a JSON payload (for API-based creation), string escaping is multi-layered. Use raw strings or write code to files to avoid shell interpolation issues.

### G16: Always Wrap Logic in try/except

The V7 Go skill sandbox does not surface tracebacks. Unhandled exceptions produce generic "skill execution error" messages that are impossible to debug. Wrap every logical step in try/except and return structured error dictionaries:

```python
import traceback

def run(...):
    try:
        # your logic
        return {"status": "success", "result": data}
    except Exception as e:
        return {
            "status": "error",
            "error": str(e),
            "traceback": traceback.format_exc()
        }
```

Always check HTTP status codes before calling `.json()`. Always include `traceback.format_exc()` and response text snippets in error returns.

### G17: No Filesystem Access

Skills run in a sandboxed environment (Sandcastle) that cannot create or use temporary files. `tempfile.mktemp()`, `tempfile.NamedTemporaryFile()`, and writing to `/tmp` will all fail. All file generation must happen entirely in memory using `BytesIO`.

### G18: Loading Binary Assets from Secrets

If a skill needs binary assets (like a logo image), store the base64-encoded data as a workspace secret and decode at runtime:

```python
import base64

def run(logo_b64: Annotated[str, Secret("LOGO_B64")]):
    logo_bytes = base64.b64decode(logo_b64)
```

Do not rely on co-located files -- the sandbox does not guarantee supporting files are available on the filesystem.

### G47: Large Inputs Get Truncated

When passing large lists through LLM-mediated properties, the data can be silently truncated before reaching the skill (observed: 2,060 items reduced to 16). Workarounds: combine producer and consumer into one skill, cap output to under 100 items, or persist intermediate state via API writes.

### G48: LLM Properties Can Invoke Skills Multiple Times

An LLM-mediated property may call a skill multiple times within a single execution. If the skill creates records, this can produce duplicates. Make writes idempotent: look up by a stable business key before creating, and convert to an update if the record already exists.

### G49: LLM Safety Filters Can Block Skill Execution

The AI model may refuse to execute a skill if the skill description or argument names trip its safety filter (observed with `gpt_5_mini`). If this happens, bypass the LLM by using the Execute Skill API directly:

```
POST /api/workspaces/{wid}/skills/{skill_id}/runs
{"params": {"arg_name": {"value": "...", "value_format": "default"}}}
```

Secret parameters annotated with `Secret()` are auto-injected -- do not pass them. Note the 60-second gateway timeout.

### G56: Prefer V7Go.call_skill Over LLM-Hosted Invocation

`V7Go.call_skill()` removes the AI model from the dispatch path entirely -- no token cost, no hallucination risk, deterministic execution. Use it whenever arguments are known at design time. Only use LLM-hosted invocation when the skill must fire based on natural-language judgment, or for file-output skills (which require an LLM host).

### G57: Match the Consumer to the Host Type

A `V7Go.call_skill` property stores its result as the plain field value. Read it with `@<P{id}>` and index into the dictionary. An LLM-hosted skill property stores results in the legacy `tool_metadata.skill_results` path. Using the wrong access pattern returns nothing.

### G58: Reading File Fields from Skills

When a skill needs to read a `data`/`file` field, the value in `manual_value.value` is a pre-signed S3 URL (a string). Fetch it WITHOUT the V7 Go API key -- the URL carries its own authentication. Extract the filename from the URL's `response-content-disposition` parameter. Include the storage host (e.g., `s3.eu-west-1.amazonaws.com`) in `allowed_domains`, or the download will return an HTML error page instead of the file.

---

## API Reference

| Action | Endpoint |
|--------|----------|
| List skills | [skill-list](https://docs.go.v7labs.com/reference/skill-list) |
| Create skill | [skill-create](https://docs.go.v7labs.com/reference/skill-create) |
| Update skill | [skill-update](https://docs.go.v7labs.com/reference/skill-update) |
| Get skill | [skill-get](https://docs.go.v7labs.com/reference/skill-get) |
| Skills documentation | [Skills](https://docs.go.v7labs.com/docs/skills-1) |
