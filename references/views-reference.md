# V7 Go Views Reference

Views are one of the most powerful features in V7 Go — and one of the most misunderstood. This reference covers what they are, how they work, and the patterns that make complex workflows possible.

---

## What Are Views?

A **view** is a filtered, sorted perspective of your workflow's data. Think of it like a saved filter in a spreadsheet: same underlying data, different lens.

Every workflow (agent) starts with a **main view** that shows all entities (submissions) and all top-level properties. You create additional views to:

- **Route different document types** to different processing steps
- **Show only relevant properties** for a specific stage of review
- **Control which AI extractions run** for which documents
- **Create review queues** for human oversight
- **Separate completed items** from work in progress

**Key concept:** Views aren't just for display. In V7 Go, views control which properties actually compute. A property placed in a view only runs for entities that match that view's filter. This is the foundation of conditional workflows.

---

## Why Views Matter

Without views, every property runs for every entity. That means:
- Every AI extraction runs on every document, even when it doesn't apply
- Users see every intermediate processing step, cluttering their workspace
- There's no way to create branching logic (do X for invoices, do Y for contracts)
- No way to separate "needs review" from "ready to process" from "completed"

Views solve all of these by creating **scoped processing lanes**. Each view defines which entities it applies to (via filters) and which properties are visible (and therefore compute) within it.

---

## Core Concepts

### The Main View

Every workflow has a **main view** that is always active for every entity. Properties on the main view run for all submissions. This is where you put:

- The file upload property
- Top-level routing properties (e.g., document type selector)
- Final output properties that every entity needs
- Summary/status properties

**Rule of thumb:** Keep the main view minimal. Only properties that every user needs to see and that apply to every entity belong here.

### Routed Views (Filtered Views)

Every view besides main is a **routed view**. A routed view has a filter that determines which entities it applies to. The filter is typically based on a `single_select` routing property.

**How routing works:**
1. An entity arrives and the routing property computes (e.g., "Document Type" is classified as "Invoice")
2. The view filtered on `Document Type = "Invoice"` activates for that entity
3. All properties in that view compute for that entity
4. Properties in other type-specific views (e.g., "Contract Processing") are skipped

**Important:** A routed view with empty filters (`filters: []`) computes for NO entities. The view must filter on a routing property to be active. There's no toggle or "activate" button — routing is entirely controlled by view filters.

### Routing Properties

A **routing property** is typically a `single_select` property whose value determines which views activate for an entity. Common routing properties include:

| Routing Property | Purpose | Example Options |
|-----------------|---------|-----------------|
| Document Type | Route by what kind of document it is | Invoice, Contract, Statement, Receipt |
| Processing Stage | Route by workflow stage | Intake, Processing, Review, Complete |
| Result / Status | Route by outcome | STP (straight-through), Human Review |
| Priority | Route by urgency | High, Medium, Low |

You can have multiple routing properties for multi-dimensional routing (e.g., route by type AND by priority).

---

## How to Create Views

### Via the V7 Go UI

1. Open your workflow (agent)
2. Click the **+** button next to existing view tabs
3. Name the view
4. Add filters (typically: select a routing property and pick which option value this view should match)
5. Choose which properties are visible in this view

### Via the API / MCP

Create a view with a POST request that includes:
- `name`: The view's display name
- `filters`: Array of filter objects that determine which entities appear
- `property_ids`: Array of property UUIDs to show in this view
- `property_layouts`: Array (can be empty `[]`) for column layout configuration

**View filter format** (for select-based routing):
```json
{
  "filters": [
    {
      "property_id": "<routing_property_id>",
      "select_option_value": "Invoice"
    }
  ]
}
```

Multiple filters are ANDed together — an entity must match ALL filters to be in the view.

> **Gotcha (G31):** View filters use `{"property_id": "...", "select_option_value": "..."}` format, NOT the nested `subject`/`matcher` structure used for entity listing. These are different endpoints with different filter formats.

> **Gotcha (G32):** View updates always require both `property_ids` and `property_layouts`, even when you're only changing filters.

---

## The `default_view_id` Pattern (G72)

When you create a new property, it automatically lands in the **main view** unless you tell it otherwise. This is one of the most common mistakes in V7 Go workflow building.

### The Problem

Intermediate processing properties (extraction steps, format converters, internal calculations) clutter the main view. Users see columns they don't need, and worse, these properties compute for every entity instead of only the ones they apply to.

### The Solution

When creating any property that doesn't belong in the main view, include `default_view_id` in the create request:

```json
{
  "name": "Invoice Line Items",
  "type": "json",
  "tool": "gemini_2_5_pro",
  "description": "Extract line items from invoice...",
  "default_view_id": "<invoice_processing_view_id>"
}
```

This places the property ONLY in the specified view, not in main.

### Cleanup: Removing Properties from Main

If a property has already leaked into the main view:
1. Update the main view to remove that property's ID from its `property_ids` array
2. Add the property to the correct processing view
3. Re-read the main view and confirm only intended properties remain

Adding a property to another view does NOT remove it from main. You must explicitly remove it.

---

## View-Scoped Computation (G73)

This is the mechanism that makes conditional workflows work. Understanding it is essential for building anything beyond simple linear workflows.

### How It Works

| View Type | Activation | Properties Compute When... |
|-----------|-----------|---------------------------|
| **Main view** | Always active for every entity | Always — every entity triggers main-view properties |
| **Routed view** | Active only when the routing filter matches | Only when the entity's routing value matches the filter |

### The Routing Chain

1. **Routing property computes** (usually on the main view): determines the entity's type/category
2. **View filter matches**: the platform checks which views filter on that routing value
3. **View-scoped properties activate**: properties in matching views begin computing
4. **Non-matching views are skipped**: properties in views that don't match are marked `inactive_view`

### What Happens to Skipped Properties

When an entity doesn't match a view's filter:
- All properties in that view are marked as **skipped** (`inactive_view`)
- They never compute — no cost, no processing time
- Downstream properties that depend on skipped properties also don't run
- This is expected behavior, not an error

### Recreating a Routing Property Breaks View Filters

If you delete and recreate a routing property (giving it a new ID), every view that filtered on the old ID silently reverts to empty filters. This means those views stop computing for ALL entities. After recreating a routing property, you must re-set every dependent view's filter to use the new property ID.

### Checking for Routing Issues

After creating or moving properties, recalculate and check the response for `ignored_reasons`. If you see `"reason_code": "inactive_view"`, the entity isn't routed to that view. This could mean:
- The routing property hasn't computed yet
- The routing value doesn't match the view's filter
- The view's filter references an old/deleted routing property

---

## Common View Patterns

### Pattern 1: Intake / Triage View

**Purpose:** Initial processing that applies to every submission.

| Properties | Notes |
|-----------|-------|
| File (upload) | Where documents enter the workflow |
| Document Type (single_select) | Routing property — classifies the document |
| Priority (single_select) | Optional routing by urgency |
| Status (single_select) | Overall workflow status |

**Filter:** None (this IS the main view, or a view with an always-matching filter).

**Tip:** If you need a processing view that runs for every entity (not just main), route it on an always-true `single_select` that defaults to `["Ready"]`.

---

### Pattern 2: Type-Specific Extraction Views

**Purpose:** Run different AI extractions based on document type.

**Example: An invoice vs. contract workflow**

| View Name | Filter | Properties |
|-----------|--------|-----------|
| Invoice Processing | `Document Type = "Invoice"` | Invoice Header Extraction, Line Item Extraction, Tax Calculation |
| Contract Processing | `Document Type = "Contract"` | Contract Parties, Key Terms, Dates & Deadlines, Obligations |
| Statement Processing | `Document Type = "Statement"` | Account Summary, Transaction List, Balance Reconciliation |

Each view contains only the extraction properties relevant to that document type. When a document is classified as an "Invoice," only the invoice extraction properties run. Contract and statement extractions are skipped (no cost, no wasted processing).

---

### Pattern 3: Review / QA View

**Purpose:** Queue items that need human attention.

| Properties | Notes |
|-----------|-------|
| Original File | For reference |
| Extracted Data (read-only display) | What the AI found |
| Confidence Score | How confident the extraction is |
| Review Notes (manual text) | For reviewer comments |
| Approval Status (single_select) | Approved / Rejected / Needs Revision |

**Filter:** `Result = "Human Review"` or `Confidence < threshold`

This view shows only items that the AI flagged for human review, with the relevant context for making a decision.

---

### Pattern 4: Completed / STP View

**Purpose:** Show items that passed through successfully (straight-through processing).

| Properties | Notes |
|-----------|-------|
| Original File | For reference |
| Final Output | The validated, merged result |
| Processing Date | When it was processed |
| Destination Status | Whether it was sent downstream |

**Filter:** `Result = "STP"` or `Status = "Complete"`

This view shows the "done" pile — items that passed validation and are ready for (or have been sent to) downstream systems.

---

### Pattern 5: Needs Review View

**Purpose:** Catch items where something went wrong or needs attention.

| Properties | Notes |
|-----------|-------|
| Original File | For reference |
| Error Details | What went wrong |
| Last Processing Attempt | When it was last tried |
| Retry Action (manual trigger) | Button to re-process |

**Filter:** `Status = "Error"` or `Status = "Needs Review"`

This is distinct from the QA view (Pattern 3). QA is for items that processed successfully but need human verification. This view is for items that hit errors or edge cases during processing.

---

## The 4-Stage View Pipeline (LLM-as-a-Judge Pattern)

The most sophisticated view pattern uses four stages to maximize extraction accuracy. This is the **LLM-as-a-Judge** pattern, where two AI models extract independently, a third model judges the results, and items are routed based on confidence.

### Stage 1: Intake & Classification

**View:** Main view (runs for every entity)

| Step | What Happens |
|------|-------------|
| File upload | Document enters the workflow |
| Document Type classification | A fast, cheap model classifies the document type |
| Routing | The classification value activates the appropriate extraction view |

### Stage 2: Type-Specific Dual Extraction

**Views:** One per document type (e.g., "Type A Extraction," "Type B Extraction")

| Step | What Happens |
|------|-------------|
| Extraction 1 (Model A) | First model extracts to a structured schema |
| Extraction 2 (Model B) | Second model (different provider) extracts to the SAME schema |

Using two different model providers maximizes the chance of catching errors — they have different failure modes, so disagreement flags ambiguity.

### Stage 3: Judging & Resolution

**View:** A view that receives all extraction outputs (or the main view)

| Step | What Happens |
|------|-------------|
| Judge property | A strong reasoning model compares both extractions against the source document |
| Self-healing | Where the judge is confident, it resolves disagreements automatically |
| Status assignment | `all_passed` (agreement or confident resolution) or `needs_review` (unresolved disagreement) |
| Result routing | A `single_select` maps status to `"STP"` or `"Human Review"` |

The judge sees the original file and can arbitrate between the two extractions, not just pick one blindly.

### Stage 4: Terminal Views

**Views:** STP view and Human Review view

| View | Filter | Purpose |
|------|--------|---------|
| STP | `Result = "STP"` | Items that passed — ready for downstream system integration |
| Human Review | `Result = "Human Review"` | Items that need a person to verify, with judge diagnostics visible |

### Visual Flow

```
Main View (all entities)
  |
  ├── File Upload
  ├── Document Type Classification
  |
  ├── [Type A View] ──── Extraction 1 (Model A) + Extraction 2 (Model B)
  ├── [Type B View] ──── Extraction 1 (Model A) + Extraction 2 (Model B)
  |
  ├── Judge (compares extractions against source)
  ├── Result (STP or Human Review)
  |
  ├── [STP View] ──── Final output, integration/export
  └── [Human Review View] ──── Review fields, judge diagnostics
```

### Key Design Decisions

- **Extractions are view-scoped:** Type A extractions never run for Type B entities (saves cost and avoids wrong prompts)
- **The Judge sits on a broad view** so it receives all extraction inputs. Inputs from inactive branches are null, so the judge only evaluates the populated pair
- **Result and global post-judge fields** sit on the main view so they run for every entity after the judge completes
- **Enable citations on the Judge property** so reviewers can trace flagged values back to specific locations in the source document

---

## Tips & Best Practices

1. **Keep main minimal.** Only top-level inputs, routing properties, and final outputs belong on the main view. Everything else goes in a routed view via `default_view_id`.

2. **Name views clearly.** Use descriptive names that tell users what they'll find: "Invoice Processing," "Pending Review," "Completed - Ready for Export."

3. **One routing property per routing dimension.** Don't overload a single select with both type AND status. Use separate routing properties for separate concerns.

4. **Test routing with a single entity first.** After setting up views, submit one test entity, let it route, and verify that the correct view's properties computed and the others were skipped.

5. **Check `ignored_reasons` after recalculation.** If properties aren't computing, the recalculation response tells you why. Look for `inactive_view` to diagnose routing issues.

6. **Remember G38:** Select values in code properties are lists. When checking routing values in code, compare with `["STP"]`, not `"STP"`.

7. **Aggregation properties go on unfiltered views.** If you need to merge results from multiple branches, the aggregation property must be on a view with no filter (typically main) so it runs for all entities.

8. **Document your view structure.** For complex workflows, maintain a registry of which views exist, what they filter on, and which properties they contain. This saves hours of debugging later.
