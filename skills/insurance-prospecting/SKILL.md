---
name: insurance-prospecting
description: >
  Prospect for commercial insurance and employee benefits accounts using
  Zywave's market intelligence database. Find target companies by industry
  (NAICS), geography, employee count, revenue, incumbent broker, renewal
  month, or compliance posture. Retrieve decision-maker contacts for
  identified prospects. Use when the user wants to find new accounts to
  pursue, displace an incumbent broker, or build a target list.
  Also use for research briefs on a specific prospect company.
allowed-tools: [mcp__zywave__discovery_companies_search, mcp__zywave__discovery_company_contacts_get, mcp__zywave__discovery_household_contacts_get, mcp__zywave__research_brief_generate, mcp__zywave__research_brief_get]
---

# Insurance Prospecting with Zywave

## Tool selection guide

| Goal | Tool |
|---|---|
| Find P&C or benefits prospects | `discovery_companies_search` |
| Preview or get contacts for a prospect | `discovery_company_contacts_get` |
| Get personal lines household contacts | `discovery_household_contacts_get` |
| Generate a full AI research brief on a prospect | `research_brief_generate` |
| Retrieve a previously generated research brief | `research_brief_get` |

Do **not** use these tools to look up accounts the user already manages — use `account_search` from the insurance-crm skill instead.

---

## discovery_companies_search

### Required parameter

`lineOfBusiness` is required and must be exactly `"Commercial"` or `"Benefits"`. There is no combined search — run two calls if both LOBs are needed.

### Key filters and how to use them

**Geography** — `states` (array of 2-letter codes: `["WI", "IL"]`) and/or `city` (string).

**Industry** — `naicsCodes` (array of strings). Pass full 6-digit codes (e.g. `"541512"`); the API filters on the first 2 digits so passing a specific code is fine and preferred.

**Size** — `employeeCountMin` / `employeeCountMax` (integers). `revenueMin` / `revenueMax` (integers in dollars, e.g. `10000000` for $10M).

**Incumbent broker** — `commercialBrokerName` or `benefitsBrokerName` (partial string match). Pass competitor names here to find displacement targets: `"Marsh"`, `"AON"`, `"M3"`.

**Renewal timing** — `renewalMonths` (array of integers 1–12). Combine with `states` for highest-conversion prospecting.

**Compliance flags** — `hasDotViolations`, `hasOshaViolations`, `fidelityBondOutOfCompliance` (booleans).

**Contact availability filter** — `hasContactEmails: true` restricts results to companies with email data available. Useful before running enriched contact lookups to avoid wasted calls.

**Pagination** — use `pageToken` from `NextPageToken` in the response to page through results. `pageSize` max is 25.

### New response fields (post-revision)

- `latitude` / `longitude` — coordinates for proximity follow-up queries
- `isOutOfBusiness` — filter these out before presenting to the user
- `qualityScore` — data confidence score; surface when prioritizing targets
- `hasContactEmails` — preview flag before calling `discovery_company_contacts_get`

### Typical prospecting workflow

1. Call `discovery_companies_search` with `lineOfBusiness` + geography + NAICS + size filters
2. Filter out `isOutOfBusiness: true` results
3. Sort by `revenueRange` and `leadCommercialBroker` to identify best displacement targets
4. For top targets, call `discovery_company_contacts_get` with `msid` and `enrich: false` first to preview contacts
5. Confirm with user before calling with `enrich: true` — enrichment is billed per company

---

## discovery_company_contacts_get

### ⚠️ Cost-gated — always preview before enriching

This tool has two modes controlled by the `enrich` parameter:

**`enrich: false` (default, no charge)** — Returns contact names, job titles, and availability flags (`HasEmail`, `HasDirectPhone`, `HasMobilePhone`). Use this to preview what's available.

**`enrich: true` (billed per company)** — Returns actual email addresses and phone numbers. This incurs a charge billed to the agency. **Always confirm with the user before setting `enrich: true`.** Never enrich silently.

### Lookup methods (in priority order)

1. `msid` — fastest; pass directly from `discovery_companies_search` results
2. `ein` + `name`
3. `name` + `city` + `state`

```
// Correct — preview first
discovery_company_contacts_get({ msid: "M84000062336619", enrich: false })
// Then, after user confirms they want contact details:
discovery_company_contacts_get({ msid: "M84000062336619", enrich: true })
```

Use `pageSize` (max 25) and `pageToken` for companies with many contacts.

---

## research_brief_generate / research_brief_get

Generates a comprehensive AI research brief for a specific prospect company. Requires the company `msid` and `lob` (`"Commercial"`, `"Benefits"`, or `"PersonalLines"`).

- Runs **synchronously** — do not call `research_brief_get` while generation is in progress
- Progress notifications include a `publicId` — store this if the user wants to retrieve the brief later
- `research_brief_get` is only for retrieving briefs from **prior sessions** using a saved `publicId`

```
// Generate a new brief
research_brief_generate({ msid: "M84000062336619", lob: "Commercial" })

// Retrieve a previously generated brief (prior session only)
research_brief_get({ publicId: "uuid-from-prior-session" })
```

---

## Tips

- Always check `isOutOfBusiness` before presenting results — filter these silently
- `qualityScore` of 75 is the baseline; higher scores indicate more complete data records
- For the displacement play: filter by `commercialBrokerName` with a competitor, then sort by `revenueRange.max` descending to find the highest-value accounts to pursue
- `hasContactEmails: true` in the search filters to companies where enrichment will actually return email data — use this to improve enrichment yield before spending credits
