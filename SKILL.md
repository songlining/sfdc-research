---
name: sfdc-research
description: Use when user asks to research Salesforce accounts, contacts, opportunities, deals, contracts, or needs to run SOQL queries against a Salesforce org
---

# Salesforce Research

Research Salesforce accounts, contacts, opportunities, and deals using the `sf` CLI. Never guess field names — describe objects first, or use known fields from this skill.

## Prerequisites & Auto-Setup

Before doing anything else, run these checks **every session**. Fix any failures automatically — don't ask the user unless auth is needed.

### Step 1: Check & Install Node.js

```bash
node --version 2>/dev/null || echo "MISSING"
```

If missing, install it:
- **macOS:** `brew install node`
- **Linux (apt):** `curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash - && sudo apt-get install -y nodejs`
- **Linux (yum):** `curl -fsSL https://rpm.nodesource.com/setup_lts.x | sudo bash - && sudo yum install -y nodejs`

Verify after install: `node --version` (need v18+).

### Step 2: Check & Install Salesforce CLI (`sf`)

```bash
sf --version 2>/dev/null || echo "MISSING"
```

If missing, install it:
```bash
npm install -g @salesforce/cli
```

Verify after install: `sf --version`.

### Step 3: Check & Install jq (for JSON processing)

```bash
jq --version 2>/dev/null || echo "MISSING"
```

If missing, install it:
- **macOS:** `brew install jq`
- **Linux (apt):** `sudo apt-get install -y jq`
- **Linux (yum):** `sudo yum install -y jq`

### Step 4: Verify Salesforce Authentication

```bash
sf org list --json 2>/dev/null | jq '.result.nonScratchOrgs[] | select(.username != null) | .username' 2>/dev/null
```

If the target org is not listed or the command fails:
1. Tell the user: "You need to log in to Salesforce. I'll open a browser window for you."
2. Run: `sf org login web --alias larry.song@hashicorp.com --instance-url https://hashicorp.my.salesforce.com`
3. Wait for the user to complete the browser login flow.
4. Verify: `sf org display --target-org larry.song@hashicorp.com --json | jq '.result.connectedStatus'`

If auth has expired (status is not "Connected"):
- Run the same login command above and walk the user through it.

### Step 5: Quick Smoke Test

Run a lightweight query to confirm everything works end-to-end:
```bash
sf data query --query "SELECT Id FROM Account LIMIT 1" --target-org larry.song@hashicorp.com --result-format json
```

If this succeeds, print: "All prerequisites verified. Ready to research."
If this fails, check the error message and troubleshoot (auth expired, network issue, etc.).

### Setup Summary

| Tool | Required | Install Command (macOS) |
|------|----------|------------------------|
| Node.js v18+ | Yes | `brew install node` |
| Salesforce CLI (`sf`) | Yes | `npm install -g @salesforce/cli` |
| jq | Recommended | `brew install jq` |
| Salesforce Auth | Yes | `sf org login web --alias larry.song@hashicorp.com` |

Default org: `larry.song@hashicorp.com`. Override per session: user says "use org X" → replace `--target-org larry.song@hashicorp.com` with `--target-org X`.

## Bootstrap: Start of Every Session

1. **Run prerequisite checks above** — install/fix anything missing automatically
2. Read `learnings.md` in this skill's directory (if it exists)
3. Merge learned fields/queries with the Known Objects below — learnings take precedence if they conflict
4. Use combined knowledge to skip unnecessary `sf sobject describe` calls

## Core Commands

```bash
# List all objects in the org
sf sobject list --target-org larry.song@hashicorp.com

# Describe an object (fields, relationships) — pipe through python/jq to filter
sf sobject describe --sobject Account --target-org larry.song@hashicorp.com --json

# Run SOQL query (table output)
sf data query --query "SELECT Id, Name FROM Account LIMIT 10" --target-org larry.song@hashicorp.com

# Run SOQL query (JSON output — better for complex/nested data)
sf data query --query "SELECT Id, Name FROM Account" --target-org larry.song@hashicorp.com --result-format json
```

## Known Objects & Key Fields

### Account
```
Id, Name, Industry, AnnualRevenue, ARR__c, Current_ACV__c,
BillingCountry, BillingCity, Account_Tier__c, Account_Sub_Segment__c,
Customer_Stage__c, Current_Products__c, Count_of_Current_Products__c,
OwnerId, Account_Owner_Email__c, CSM_Full_Name__c, CSM_Sentiment__c,
Customer_Pulse__c
```

### Opportunity
```
Id, Name, StageName, Amount, ACV__c, New_ACV__c, CloseDate, Type,
Primary_Product__c, Product_Families__c, Product_s_Targeted__c,
ForecastCategoryName, Probability, OwnerId, Sell_Type__c,
Transaction_Type__c, IsClosed
```
Filter open opps: `WHERE IsClosed = false`

### Contract
```
Id, ContractNumber, Name, Status, StartDate, EndDate, ContractTerm,
SBQQ__ActiveContract__c, CurrencyIsoCode, SBQQ__Opportunity__c, AccountId
```
Filter active contracts: `WHERE Status = 'Activated'`

### SBQQ__Subscription__c (Contract Line Items)
```
SBQQ__ProductName__c, Product_Line__c, Product_Family__c, Product_Edition__c,
SBQQ__Quantity__c, Unit_Value__c, Total_Entitlement_Quantity__c,
SBQQ__NetPrice__c, Net_Total__c, SBQQ__StartDate__c, SBQQ__EndDate__c,
SubscriptionTerm__c, SBQQ__Contract__c
```
Links to Contract via `SBQQ__Contract__c`.

## Guided Research Workflow

When user asks to "research [company]" or similar open-ended request:

```
1. Find Account
   → SELECT Id, Name FROM Account WHERE Name LIKE '%keyword%'

2. Account Details
   → SELECT Id, Name, Industry, AnnualRevenue, ARR__c, Current_ACV__c,
       BillingCountry, BillingCity, Account_Tier__c, Current_Products__c,
       Account_Owner_Email__c, CSM_Full_Name__c, CSM_Sentiment__c
     FROM Account WHERE Id = 'xxx'

3. Contacts
   → Describe Contact first (or check learnings.md), then query by AccountId

4. Opportunities
   → SELECT Id, Name, StageName, Amount, ACV__c, New_ACV__c, CloseDate,
       Type, Product_Families__c, ForecastCategoryName, Probability
     FROM Opportunity WHERE AccountId = 'xxx' AND IsClosed = false
     ORDER BY CloseDate ASC

5. Active Contracts
   → SELECT Id, ContractNumber, Status, StartDate, EndDate, ContractTerm,
       SBQQ__ActiveContract__c
     FROM Contract WHERE AccountId = 'xxx' AND Status = 'Activated'
     ORDER BY EndDate DESC

6. Contract Line Items (Products/SKUs)
   → SELECT SBQQ__ProductName__c, Product_Line__c, Product_Family__c,
       Product_Edition__c, SBQQ__Quantity__c, Unit_Value__c,
       Total_Entitlement_Quantity__c, SBQQ__NetPrice__c, Net_Total__c,
       SBQQ__StartDate__c, SBQQ__EndDate__c, SubscriptionTerm__c
     FROM SBQQ__Subscription__c WHERE SBQQ__Contract__c = 'contractId'
     ORDER BY SBQQ__ProductName__c
```

At each step:
- Present a summary of findings in a readable table
- Ask the user what to drill into next
- For objects not listed above, describe first before querying

## The Rule: Describe Before Query

For objects NOT listed in "Known Objects & Key Fields" above and NOT in `learnings.md`:

1. Run `sf sobject describe --sobject <Object> --target-org larry.song@hashicorp.com --json`
2. Identify queryable fields and relationships
3. Then build the SOQL query with known field names

**Never assume field names exist.** Custom orgs have custom fields. `SELECT *` does not work in SOQL.

## SOQL Quick Reference

| Pattern | Example |
|---------|---------|
| String filter | `WHERE Name = 'Acme'` (single quotes) |
| Fuzzy match | `WHERE Name LIKE '%acme%'` (case-insensitive) |
| Date filter | `WHERE CloseDate > 2025-01-01` (no quotes) |
| Related records | `SELECT Name, (SELECT Name FROM Contacts) FROM Account` |
| Cross-object | `SELECT Name, Account.Name FROM Contact` |
| Limit results | `LIMIT 50` (always add unless user wants all) |
| Count | `SELECT COUNT() FROM Opportunity WHERE StageName = 'Closed Won'` |

## On-Demand Query Mode

When user asks a specific question, translate directly to SOQL:
- Use known fields from this skill and learnings.md
- For unknown objects/fields, describe first
- "How many open opps over $100k?" → query Opportunity
- "Who are the contacts at Acme?" → find account, then query contacts

## Error Handling

- **"No such column" error:** Describe the object, find the correct field name, retry
- **Too many results:** Add `LIMIT 50` and tell the user the total count if possible
- **Timeout on large queries:** Narrow the WHERE clause or add LIMIT
- **Auth errors:** Tell user to run `sf org login web --alias larry.song@hashicorp.com`

## Self-Improvement

This skill improves itself through two mechanisms:

### 1. Learnings File (learnings.md)

Append-only file in this skill's directory. Captures runtime discoveries.

**When to write to learnings.md — do this automatically, no need to ask:**
- A new object was described and useful fields were identified
- A query pattern worked well and is likely reusable
- A relationship between objects was discovered (e.g., "OpportunityLineItem links to PricebookEntry")
- An unexpected field name or org-specific gotcha was encountered
- A query failed and the fix was non-obvious

**Format:**
```markdown
## [Object Name] (YYYY-MM-DD)

**Useful fields:** Field1, Field2, Custom_Field__c
**Relationships:** ChildObject via ParentId, RelatedList via lookup
**Working queries:**
- `SELECT ... FROM ... WHERE ...` — description of what this finds
**Gotchas:** Any surprises or things that didn't work as expected
```

**Rules:**
- Append only — never delete or rewrite existing entries
- Don't duplicate what's already in learnings.md or in Known Objects above
- Date-stamp each entry so stale info can be identified later

### 2. Skill File Updates (this file)

When a pattern has been validated across multiple sessions (appears in learnings.md 3+ times or confirmed reliable):

**Promote to SKILL.md — ask the user first:**
- Add the object to "Known Objects & Key Fields" with its proven fields
- Add proven query patterns to the Guided Research Workflow
- Add new error patterns to Error Handling
- Remove the corresponding learnings.md entries to avoid duplication

This creates a feedback loop:
```
Session work → learnings.md (automatic)
              → SKILL.md promotion (after validation, with user approval)
              → faster future sessions
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Guessing field names | Use known fields above, check learnings.md, or `sf sobject describe` first |
| Forgetting `--target-org` | Include on every command |
| Using double quotes in SOQL strings | SOQL uses single quotes for strings |
| Quoting dates | Dates are unquoted: `CloseDate > 2025-01-01` |
| No LIMIT on exploratory queries | Default to `LIMIT 50` |
| Selecting too many fields | Start with Id, Name, and key fields; expand as needed |
| Not saving discoveries | Write to learnings.md automatically after describing new objects or finding useful patterns |
