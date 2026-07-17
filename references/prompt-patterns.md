# V7 Go Prompt Engineering Patterns

Reusable prompt templates for V7 Go agent properties. Use `@<Pproperty_ID>` syntax to reference other properties.

## Pattern 1: Basic Extraction

For extracting a single field from a document. Simple, direct, low token cost.

```
You are an expert [domain specialist]. Based on the provided document @<Pproperty_[INPUT_ID]>, extract the [field name].

[Specific extraction instructions — what to look for, where it typically appears]

You must respond with the [field name] only. If not found, respond with "N/A".
```

**When to use**: Single-field extraction, entity identification, date/number extraction
**Recommended tool**: `gpt_5` or `gpt_4o` with `thinking_effort: "disabled"` or `"low"`
**Grounded**: Yes

### Production Example (Insurance Slip — Underwriter)

```
You are an expert in the insurance industry, specifically in underwriting. Based on the provided document, you must extract the underwriter. This is the individual or entity responsible for evaluating and assuming the risk of insuring clients. Identified on the slip in the Security Details section. You must respond with the name of the underwriter only. If no underwriter is provided in the document, then you must respond with "N/A".
```

## Pattern 2: Classification

For routing documents into categories using single_select properties.

```
You are an expert in [domain]. Your task is to classify the provided document as one of [option1, option2, option3].
* [Option1] — [description of what qualifies]
* [Option2] — [description of what qualifies]
* [Option3] — [description of what qualifies]

If the document does not fit into any of these categories, classify it as [fallback option].
```

**When to use**: Document routing, type detection, status assignment
**Recommended tool**: `gpt_5_2` or `gpt_5_mini` with `thinking_effort: "disabled"`
**Grounded**: No

### Production Example (Insurance Slip — Classification)

```
You are an expert in the insurance industry. Your task is to classify the provided document as one of slip, endorsement or other.
* A slip will layout the terms by which an insurance policy will be written.
* An endorsement will set out the change or adjustment to an existing insurance policy.
If the document does not fit into either of these 2 categories then you must classify it as other.
```

## Pattern 3: Reasoning / Analysis

For multi-step analysis that requires context, judgment, and structured output.

```
## Context
You are a [role] [performing task] for [organization/team] on [subject matter].

## Task
[Clear objective statement — what to extract, analyze, or evaluate]

Topics to cover:
- [Topic 1 with specifics]
- [Topic 2 with specifics]
- [Topic 3 with specifics]

### Exclusions
- If data are not disclosed, state "N/A". Preserve any qualifiers (~, +, >). Ensure units ($, %, CAGR) and time basis are explicit.
- [Other exclusions or constraints]

## Writing Style
- [Tone and format instructions]
- No overhyped marketing terminology
- [Length/format constraints]

## Output Format
[Specific format: bullets, markdown headers, JSON, etc.]

## Sources
[How to cite sources if applicable]
```

**When to use**: Investment analysis, market research, multi-section extraction, synthesized output
**Recommended tool**: `gpt_5` or `gpt_5_1` with `thinking_effort: "medium"` or `"high"`
**Grounded**: Depends — Yes if extracting from documents, No if synthesizing

### Production Example (Dataroom Screening — TAM & Projected Growth)

```
## Context
You are a professional and expert investment analyst drafting the industry overview section of a private equity screening memo for the Direct Equities team at V7 with utmost accuracy on a specific investment opportunity.

## Task
Extract and summarise TAM & Projected Growth of the **Industry** from the hub.

Topics to prioritize discussing:
- The market size (TAM and SAM), as well as the projected growth rate of each
- If the company plays in multiple industry sectors, provide market size and growth rate of each
- Break down the main drivers of future growth for this industry

### Exclusions
- If data are not disclosed, state "N/A". Preserve any qualifiers (~, +, >).
- Do NOT use the companies numbers for this, e.g. revenue growth. Focus solely in the industry.
- Amount in $B or $M e.g. $1.5B instead of 1.5 billion

## Writing Style
- Synthesise all the above information into a holistic summary using clear and direct natural language
- No overhyped marketing terminology
- No full stops at the end of each bullet point
```

## Pattern 4: Compliance Review / Redlines

For evaluating documents against guidelines and suggesting corrections.

```
Refer to the @<Pproperty_[GUIDELINES_ID]>

Does the @<Pproperty_[DOCUMENT_ID]> follow the guidelines for [section/topic]?
[Apply leniency instructions if applicable]

If so, respond with "Approved" [or "Acceptable with Caution" if guidelines state so]

If anything requires escalation, type "Needs Escalation" and suggest:
1. Which elements are non-compliant and need escalation.
2. Suggest a redline to send back to the client in order to make the section compliant.

If anything is a red flag, type "Red Flag" before the element, instead of "Needs Escalation", as a list.
```

**When to use**: Contract review, compliance checking, policy evaluation, audit
**Recommended tool**: `gpt_5` with `thinking_effort: "low"` to `"medium"`
**Grounded**: Yes

### Production Example (NDA Compliance — Entity Verification)

```
Refer to the @<Pproperty_O6E-jQXfcW1DBw1G>

Does the @<Pproperty_T7A2GEq6G-XyZP3H> follow the guidelines for Entity Verification and Document basics?

Is the Company Information correctly displayed?
Apply some lenience.

If so, respond with "Approved" (or Acceptable with Caution, if the guidelines state so)

If anything requires escalation, type "Needs Escalation" and suggest:
1. Which elements are non-compliant and need escalation.
2. Suggest a redline to send back to the client in order to make the section compliant.
If anything is a red flag, and certainly requires a redline, type "Red Flag" before the element, instead of "Needs Escalation", as a list.
```

## Pattern 5: Web Research / Enrichment

For using the web_search tool to gather and synthesize external information.

```
[Action verb] the @<Pproperty_[COMPANY_NAME_ID]> [research objective] from [source type] of its @<Pproperty_[URL_ID]>.

- [Specific research instruction 1]
- [Specific research instruction 2]
- [Specific research instruction 3]
- [What to avoid or exclude]

Return your output in Markdown with headers and bullet points. [Format instructions]
```

**When to use**: Company research, competitive analysis, market data gathering
**Recommended tool**: `web_search` with `thinking_effort: "low"`
**Grounded**: No

### Production Example (GTM Company Analysis — Value Prop)

```
Extract the @<Pproperty_0pdBGc8TIr1Of_0I> value proposition from the homepage and product pages of its @<Pproperty_Q4cff8OAkyLYKC9b>.
- Visit the homepage and primary product pages.
- Summarize the main customer benefits and positioning statements.
- Identify keywords or repeated language emphasizing their differentiators.
- Avoid internal taglines or vague marketing fluff. Focus on external value claims.
Return your output in Markdown with headers and bullet points.
```

## Pattern 6: Summary / Rollup

For aggregating outputs from multiple prior properties into a consolidated view.

```
Create a summary of [what to summarize] based on the following:
@<Pproperty_[PROP_1_ID]>
@<Pproperty_[PROP_2_ID]>
@<Pproperty_[PROP_3_ID]>

[Specific aggregation instructions — what to highlight, how to organize]
```

**When to use**: Final summary, consolidated redline report, overall score justification
**Recommended tool**: `gpt_5` with `thinking_effort: "low"`
**Grounded**: Yes (if aggregating document-sourced data)

### Production Example (NDA — Summary of Redlines)

```
Create a summary of the suggested red-lines and changes that we will need to make, in order to make the NDA compliant with our @<Pproperty_O6E-jQXfcW1DBw1G>
```

## Pattern 7: Structured Data Extraction (Tear Sheet Style)

For extracting normalized, structured data with strict formatting rules.

```
## Role
You are a [role] specialized in [task] for [organization].

## Objective
Extract or calculate the requested field with precision and return output in a normalized format. Accuracy is critically important. Prioritize correctness.

# Field to extract: [Field Name]
[Specific extraction instructions]

## Formatting Rules
- Use `Not Found` when information is not located in the source documents.
- Use `Not Applicable` only when the provision truly does not apply.
- Normalize all dates to `YYYY-MM-DD` and all currencies to USD unless otherwise stated.
- Express percentages as decimal percentages (e.g. 37.20%).
- Express multiples with one or two decimal places followed by 'x' (e.g. 1.30x).
- Do not guess or fabricate numbers.
```

**When to use**: Financial data extraction, standardized reporting, tear sheets
**Recommended tool**: `gemini_2_5_flash` or `gpt_5` with `thinking_effort: "low"`
**Grounded**: Yes

## Prompt Writing Rules

1. **Be specific about the role** — "You are an expert insurance underwriting analyst" not "You are a helpful assistant"
2. **Include N/A handling** — Always tell the model what to do when data is not found
3. **Specify output format** — Markdown, JSON, plain text, bullets, etc.
4. **Reference inputs explicitly** — Use `@<Pproperty_ID>` for every input the model needs
5. **Add exclusions** — What the model should NOT do or include
6. **Keep it focused** — One property, one task. Split complex tasks across properties.
7. **Include formatting rules** — Date formats, number formats, currency, units
8. **Add leniency/strictness instructions** — "Apply some lenience" or "Be strict — flag any deviation"

## Common Pitfalls

- **Overloaded prompts** — If a prompt does 3+ things, split into separate properties
- **Missing N/A handling** — Model will hallucinate when data is missing without explicit instructions
- **Vague output format** — "Summarize" without format spec produces inconsistent results
- **No input references** — Forgetting `@<Pproperty_ID>` means the model can't access the data
- **Generic roles** — "You are a helpful assistant" wastes tokens and reduces accuracy
