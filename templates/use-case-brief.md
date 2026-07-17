# Use Case Brief: [Use Case Name]

Use this template to scope a V7 Go use case before designing the workflow. This helps you think through the full picture — who needs it, what it replaces, and what success looks like.

---

## Overview

**Use case name**: _e.g., "Lease Abstraction" or "Claims Intake Triage"_

**One-sentence description**: _What does this automate?_

**Vertical**: _Financial Services / Real Estate / Insurance / Legal / Healthcare / Other_

**Department**: _e.g., Investment Operations, Underwriting, Legal, Compliance_

---

## Current State

**How is this done today?**
_Describe the manual process step by step._

1. _e.g., Analyst receives PDF via email_
2. _e.g., Opens document and reads through 50+ pages_
3. _e.g., Manually extracts key terms into a spreadsheet_
4. _e.g., Sends spreadsheet to manager for review_

**Pain points:**
- _e.g., Takes 2 hours per document_
- _e.g., Error-prone — inconsistent data entry across analysts_
- _e.g., Bottleneck — only 3 analysts can do this work_

**Current volume**: ___ documents per week/month

**Current time per document**: ___ minutes/hours

**Current accuracy**: ___% (estimated)

---

## Document Profile

**Document types involved:**

| Document | Format | Pages | Complexity |
|----------|--------|-------|------------|
| _e.g., Limited Partnership Agreement_ | _PDF (digital)_ | _50-200_ | _High — legal language, tables, nested clauses_ |
| | | | |

**Key characteristics:**
- Digital PDF or scanned: ___
- Standardized format or varies widely: ___
- Contains tables/charts: ___
- Multi-language: ___

---

## Desired Outcome

**What does the ideal output look like?**
_Describe the end state as if it already works perfectly._

_e.g., "When I upload a batch of LPAs, V7 Go extracts all 44 key terms into a structured table within minutes. Each extraction includes a citation showing exactly where in the document the data came from. Flagged items go to a 'Needs Review' queue. Clean data flows directly into our portfolio management system."_

**Output format:**
- [ ] Structured data table (rows and columns)
- [ ] Generated document (memo, report, summary)
- [ ] Classification/triage (route to the right person/team)
- [ ] Dashboard or scorecard
- [ ] Combination: ___

---

## Data Points to Extract

_List every specific piece of information you need from the documents._

| # | Data Point | Example Value | Notes |
|---|-----------|---------------|-------|
| 1 | _e.g., Fund Name_ | _"Blackrock Growth Fund IV"_ | _Usually on first page_ |
| 2 | _e.g., Management Fee_ | _"1.5%"_ | _May be tiered — need all tiers_ |
| 3 | _e.g., Carried Interest_ | _"20%"_ | _Look for hurdle rate too_ |
| 4 | | | |
| 5 | | | |

---

## Downstream Systems

**Where does the output need to go?**

| System | What Gets Sent | How Often |
|--------|---------------|-----------|
| _e.g., Excel/Google Sheets_ | _All extracted data points_ | _After each batch_ |
| _e.g., Salesforce_ | _Deal terms summary_ | _Per document_ |
| _e.g., Email notification_ | _Flagged items requiring review_ | _Real-time_ |

---

## Success Metrics

| Metric | Current | Target |
|--------|---------|--------|
| Time per document | _e.g., 2 hours_ | _e.g., 10 minutes_ |
| Accuracy | _e.g., 85%_ | _e.g., 95%+_ |
| Volume capacity | _e.g., 10/week_ | _e.g., 100/week_ |
| Human review rate | _e.g., 100%_ | _e.g., 15-20%_ |

---

## Stakeholders

| Role | Name | Involvement |
|------|------|------------|
| Primary user | | _Who runs the workflow day-to-day?_ |
| Reviewer | | _Who validates output quality?_ |
| Decision maker | | _Who approves the workflow going live?_ |
| IT/Security | | _Who needs to approve integrations?_ |

---

## Constraints & Requirements

- [ ] Compliance requirements: _e.g., SOX, HIPAA, GDPR_
- [ ] Data residency: _e.g., Data must stay in US/EU_
- [ ] Audit trail needed: _e.g., Every extraction must be traceable to source_
- [ ] Security review required: _Yes / No_
- [ ] Timeline: _When does this need to be live?_
