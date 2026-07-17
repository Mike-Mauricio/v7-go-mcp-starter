# V7 Go Workflow Design — Best Practices

These 10 principles are distilled from dozens of production workflows built across financial services, real estate, insurance, and legal verticals. Follow them and your workflows will be more accurate, faster, and easier to maintain.

---

## 1. Start With the Output, Work Backwards

Before touching V7 Go, answer: **What does the final deliverable look like?**

Sketch the output table, document, or report you want. List every column, field, or section. Then design the workflow steps to produce exactly that — nothing more.

**Why this matters:** Most workflow problems start with vague goals. "Extract data from documents" isn't specific enough. "Extract fund name, management fee, carried interest, and term length into a table with one row per document" — that's specific enough to build from.

---

## 2. One Property, One Task

Each property in your workflow should do exactly one thing. If you find yourself writing a prompt that says "extract X and also classify Y and summarize Z" — split it into three separate properties.

**Good:**
- Property 1: Extract management fee
- Property 2: Extract carried interest
- Property 3: Classify fund type

**Bad:**
- Property 1: Extract all fee information, classify the fund, and write a summary

**Why:** Smaller, focused properties are more accurate, easier to debug, and cheaper to run (you can use simpler models for simple tasks).

---

## 3. Follow the Sequential Pipeline Pattern

Most successful workflows follow this sequence:

```
Input → Classify → Extract → Analyze → Synthesize → Status
```

1. **Input** — The document or data the user uploads
2. **Classify** — What type of document is this? (routes downstream logic)
3. **Extract** — Pull out specific data points (the bulk of most workflows)
4. **Analyze** — Compare, calculate, or assess the extracted data
5. **Synthesize** — Generate summaries, memos, or recommendations
6. **Status** — Flag for review, mark as complete, or route to next step

Not every workflow needs all six steps. A simple extraction workflow might only need Input → Extract → Status. But if you're unsure how to structure your workflow, this pattern is a safe starting point.

---

## 4. Use Selective Grounding

**Grounding** (AI Citations) traces every extracted value back to its exact source location in the original document — with visual bounding boxes on the PDF.

Turn grounding **ON** for:
- Any property that extracts specific data from a document (names, numbers, dates, clauses)
- Properties where audit trail or compliance matters

Turn grounding **OFF** for:
- Summaries, analysis, and generated text (there's no single source location)
- Classifications and status properties
- Properties that use web search, Knowledge Hubs, or code execution

**Why:** Grounding improves accuracy for extraction tasks (the AI has to point to its source), but adds unnecessary overhead for generative tasks.

---

## 5. Include N/A Handling in Every Extraction Prompt

Every prompt that extracts data from a document must include: **"If not found, respond with N/A."**

Without this, the AI may hallucinate a plausible-sounding answer when the information isn't actually in the document. With it, you get a clean signal that the data wasn't found — which is far more useful than a wrong answer.

**Example:**
> Extract the management fee percentage from @Document. Format as a percentage (e.g., 1.5%). If not found, respond with N/A.

---

## 6. Mix Model Providers by Task

Different AI models are better at different things. Don't use the same model for every property — match the model to the task:

| Task Type | Best Model | Why |
|-----------|-----------|-----|
| Fast extraction | `gemini_3_flash` | Speed + low cost |
| Classification | `gpt_5_mini` or `claude_4_5_haiku` | Good at categories |
| Structured JSON | `gpt_5_5` | Reliable formatting |
| Complex analysis | `claude_4_6_sonnet` | Strong reasoning |
| Hub search | `gpt_5_5` | Best at hub queries |
| Long summaries | `gemini_3_1_pro` | Large context |
| Compliance review | `claude_4_6_sonnet` | Rule evaluation |
| Deep reasoning | `claude_4_7_opus` | Complex code + analysis |

**Why:** Using the right model per task can cut costs by 50-80% while maintaining or improving accuracy.

---

## 7. Start Thinking Effort at Low

The `low` thinking effort level works on all models and is sufficient for most extraction and classification tasks. Only increase when needed:

- `minimal` — For trivial tasks (yes/no, simple lookup). Works with GPT-5 Mini and Gemini 3 Flash.
- `low` — Default starting point. Works on most models (not Claude).
- `medium` — For multi-step reasoning, comparisons, analysis.
- `high` — For deep reasoning, complex compliance review.

**Important exception:** Claude models (Sonnet, Opus) don't support `low` — jump from `disabled` to `medium`. For GPT-5.5, Gemini 3.1 Pro, and Claude 4.5 Haiku, check V7 Go docs for current thinking effort compatibility.

**Why:** Higher thinking effort is slower and more expensive. Start low, increase only if accuracy isn't good enough.

---

## 8. Normalize Output Formats

Tell the AI exactly how to format extracted data. Inconsistent formats make downstream analysis painful.

- **Dates:** "Format as YYYY-MM-DD"
- **Percentages:** "Format as a percentage (e.g., 1.5%)"
- **Currency:** "Format in millions (e.g., $150M)"
- **Names:** "Format as Last, First"
- **Yes/No:** "Respond with Yes, No, or N/A"

Add format instructions to every extraction prompt. Your future self will thank you.

---

## 9. Use Python Properties for Zero-Cost Logic

V7 Go has a `code` property type that runs Python — no AI model needed, so it's free and instant. Use it for:

- **Formatting:** Convert dates, calculate percentages, standardize text
- **Parsing:** Split comma-separated values, extract patterns with regex
- **Conditional logic:** If-else routing that doesn't need AI judgment
- **Calculations:** Math operations on extracted numbers

**Example:** Instead of asking an LLM to calculate "revenue growth = (current - prior) / prior," use a Python property. It's faster, cheaper, and 100% accurate.

---

## 10. Test With Representative Documents Before Scaling

Before processing 1,000 documents, test with 5-10 that represent the full range of what you'll see:

- One "clean" document (well-formatted, all fields present)
- One "messy" document (scanned, poor quality, handwriting)
- One "edge case" (unusual format, missing sections, very long)
- One from each document subtype (if your workflow handles multiple types)

Run these through, check every extracted value against the source, and refine your prompts before scaling up. The 30 minutes you spend testing saves hours of cleanup later.

---

## Quick Reference Checklist

Use this before deploying any workflow:

- [ ] Every property does exactly one thing
- [ ] Extraction prompts include "If not found, respond with N/A"
- [ ] Output formats are specified (dates, percentages, currency)
- [ ] Grounding is ON for extraction, OFF for generation
- [ ] Models are matched to task complexity (not one model for everything)
- [ ] Thinking effort starts at `low` (increased only where needed)
- [ ] Tested with 5-10 representative documents
- [ ] Status/routing property flags items that need human review
