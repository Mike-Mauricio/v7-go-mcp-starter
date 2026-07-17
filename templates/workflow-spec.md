# Workflow Spec: [Your Workflow Name]

Use this template to plan your V7 Go workflow before building it. Fill in each section — the more detail you provide, the better the AI can help you build it.

---

## Purpose

**What problem does this solve?**
_Describe in plain language what this workflow automates. Example: "Automatically extract key financial data from quarterly earnings reports and organize it into a structured table."_



**Who uses the output?**
_Who will review or act on the results? Example: "Our investment analysts use the extracted data to build financial models."_



**How is this done today?**
_What's the current manual process? Example: "Analysts manually read each PDF and type the numbers into a spreadsheet. Takes about 45 minutes per report."_



---

## Inputs

**What documents or data go into this workflow?**

| Input | Type | Example |
|-------|------|---------|
| _e.g., Quarterly earnings report_ | _PDF upload_ | _"ACME Corp Q3 2025 10-Q filing"_ |
| | | |
| | | |

**Input types available:**
- `File upload` — PDF, DOCX, images, scans
- `Text input` — Company name, question, instruction
- `URL` — Web page to analyze
- `Knowledge Hub` — Reference library (policy docs, guidelines, playbooks)

**Typical document characteristics:**
- Page count range: ___
- Structured or unstructured: ___
- Quality (digital PDF vs. scanned): ___
- Volume (how many per day/week): ___

---

## Processing Steps

**What happens between input and output?**
_List the steps in order. The AI will help you refine these into V7 Go properties._

1. **Classify**: _What categories or types need to be identified first? Example: "Determine if the report is a 10-K, 10-Q, or 8-K filing."_

2. **Extract**: _What specific data points need to be pulled out? List them._
   - _e.g., Company name_
   - _e.g., Revenue (current quarter)_
   - _e.g., Net income_
   - _e.g., EPS (diluted)_

3. **Analyze**: _Any comparisons, calculations, or assessments needed? Example: "Compare current quarter revenue to prior quarter and flag if decline exceeds 10%."_

4. **Synthesize**: _Any narrative summary or document generation? Example: "Generate a 3-paragraph executive summary highlighting key financial changes."_

5. **Route/Status**: _Any triage or routing logic? Example: "Flag as 'Needs Review' if any extracted field has low confidence."_

---

## Outputs

**What comes out of this workflow?**

| Output | Format | Description |
|--------|--------|-------------|
| _e.g., Extracted financials_ | _Structured table (AI Table)_ | _"Each row = one document, columns = extracted data points"_ |
| _e.g., Executive summary_ | _Generated text_ | _"3-paragraph summary per document"_ |
| | | |

**Output patterns** (which one fits?):
- [ ] **Unstructured → Structured data** — Documents in, organized data table out
- [ ] **Unstructured → Document** — Documents in, new document out (memo, report, Excel)
- [ ] **Human-AI augmentation** — AI handles 80-90%, human reviews the rest

---

## Integrations

**Where does the output go after V7 Go processes it?**

| Destination | Method | Details |
|-------------|--------|---------|
| _e.g., Google Sheets_ | _Zapier / MCP connector_ | _"Push extracted data to the 'Financial Data' sheet"_ |
| _e.g., Email_ | _Zapier_ | _"Send summary to analyst team"_ |
| _e.g., Salesforce_ | _MCP connector_ | _"Update deal record with extracted terms"_ |

**Integration options:**
- Zapier (8,000+ app connections)
- MCP connectors (direct connections to HubSpot, Salesforce, Slack, Google Drive, etc.)
- API (custom integrations)
- Download/export (manual)

---

## Success Criteria

**How do you know this workflow is working?**

- [ ] Accuracy target: ___% (V7 Go typically achieves 95-99%)
- [ ] Processing time target: ___ per document
- [ ] Human review rate: ___% of outputs need manual review
- [ ] Volume target: ___ documents per day/week
- [ ] Specific validation: _e.g., "Extracted revenue matches the source document within $1M"_

---

## Additional Context

**Anything else the AI should know?**
_Industry-specific terminology, edge cases, formatting requirements, compliance needs, etc._


