# Context Index

This file helps you (and your AI assistant) find the right reference material for any V7 Go task. Read this first, then load only the files relevant to your current work.

---

## Quick Task Lookup

| What You Want To Do | Read These Files |
|---------------------|-----------------|
| **Get started / first workflow** | `docs/quickstart.md`, `skills/getting-started/SKILL.md` |
| **Build a new workflow** | `skills/build-workflow/SKILL.md`, `references/prompt-patterns.md`, `references/model-selection-guide.md` |
| **Plan a workflow before building** | `skills/design-agent/SKILL.md`, `templates/workflow-spec.md` |
| **Explore existing workflows** | `skills/explore-data/SKILL.md` |
| **Choose the right model** | `references/model-selection-guide.md` |
| **Write prompts for properties** | `references/prompt-patterns.md` |
| **Understand property types** | `references/property-types-reference.md` |
| **Set up views for routing/triage** | `references/views-reference.md` |
| **Connect to external tools** | `references/integration-guide.md` |
| **Set up triggers (email, schedule, webhook)** | `references/triggers-reference.md` |
| **Build platform skills** | `references/skills-architecture.md` |
| **Handle US vs EU regions** | `references/regional-setup.md` |
| **Avoid common mistakes** | `references/gotchas.md` (start with the Top 10) |
| **Understand the platform** | `references/v7-go-platform-guide.md` |
| **See workflow design examples** | `references/workflow-patterns.md`, `references/examples/` |
| **Scope a use case** | `templates/use-case-brief.md` |
| **Find API endpoints** | `references/api-endpoints.md` |
| **Migrate/rebuild agents** | `references/advanced/rebuild-playbooks.md` |
| **Set up SharePoint integration** | `references/advanced/sharepoint-integration.md` |
| **Work across multiple workspaces** | `references/advanced/cross-workspace.md` |
| **Track workspace resources** | `templates/workspace-resources.md` |

---

## File Inventory

### Core Files (Always Loaded)

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Master instruction file — V7 Go context, best practices, behavioral defaults |
| `CONTEXT-INDEX.md` | This file — routes you to the right references |

### Skills (Guided Workflows)

| File | Purpose |
|------|---------|
| `skills/build-workflow/SKILL.md` | Step-by-step guided workflow builder |
| `skills/design-agent/SKILL.md` | Plan a workflow and produce a spec sheet (no deployment) |
| `skills/explore-data/SKILL.md` | Query and explore existing V7 Go workspace data |
| `skills/getting-started/SKILL.md` | First-time onboarding and setup |

### Reference Docs

| File | Purpose | When to Load |
|------|---------|-------------|
| `references/v7-go-platform-guide.md` | Full platform overview | When learning about V7 Go |
| `references/prompt-patterns.md` | 7 prompt templates with examples | When writing property prompts |
| `references/model-selection-guide.md` | Model-to-task matrix, thinking effort | When choosing models |
| `references/property-types-reference.md` | All property types, tools, configs | When configuring properties |
| `references/workflow-patterns.md` | 8 production workflow patterns | When designing workflow architecture |
| `references/integration-guide.md` | MCP Connectors, Zapier, API | When connecting to external tools |
| `references/views-reference.md` | Views, routing, triage patterns | When setting up views |
| `references/triggers-reference.md` | Trigger types, creation, deployment | When automating workflow runs |
| `references/skills-architecture.md` | Platform skills, V7Go.metadata, secrets | When building custom skills |
| `references/regional-setup.md` | EU vs US regions, data residency | When working with US environment |
| `references/gotchas.md` | 74 platform gotchas | When troubleshooting or before deploying |
| `references/api-endpoints.md` | Complete API endpoint reference | When making direct API calls |
| `references/examples/` | 5 production agent JSON configs | When studying real agent designs |

### Advanced References

| File | Purpose | When to Load |
|------|---------|-------------|
| `references/advanced/rebuild-playbooks.md` | Agent migration and rebuild procedures | When migrating between workspaces |
| `references/advanced/sharepoint-integration.md` | SharePoint integration architectures | When connecting to SharePoint |
| `references/advanced/cross-workspace.md` | Multi-workspace operations, workspace isolation | When managing multiple workspaces |

### Templates

| File | Purpose |
|------|---------|
| `templates/workflow-spec.md` | Template for planning a workflow |
| `templates/use-case-brief.md` | Template for scoping a use case |
| `templates/workspace-resources.md` | Template for tracking workspace UUIDs, secrets, agents |

### Docs

| File | Purpose |
|------|---------|
| `docs/quickstart.md` | 5-minute first workflow guide |
| `docs/best-practices.md` | 10 workflow design principles |
| `docs/faq.md` | Common questions and troubleshooting |
