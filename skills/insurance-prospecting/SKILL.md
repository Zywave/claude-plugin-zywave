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
allowed-tools: [mcp__zywave__discovery_company_search, mcp__zywave__discovery_company_contact_get, mcp__zywave__discovery_household_contact_get, mcp__zywave__research_brief_generate, mcp__zywave__research_brief_get]
---

# Prospecting with Zywave

## Tool selection guide

| Goal | Tool |
|---|---|
| Find P&C or benefits prospects | `discovery_company_search` |
| Preview or get contacts for a prospect | `discovery_company_contact_get` |
| Get personal lines household contacts | `discovery_household_contact_get` |
| Generate a full AI research brief on a prospect | `research_brief_generate` |
| Retrieve a previously generated research brief | `research_brief_get` |

Do **not** use these tools to look up accounts the user already manages — use `account_search` from the insurance-crm skill instead.

---

## discovery_company_search

### Required parameter

`lineOfBusiness` is required and must be exactly `"Commercial"` or `"Benefits"`. There is no combined search — run two calls if both LOBs are needed.

### Key filters and how to use them

**Geography** — `states` (array of 2-letter codes: `["WI", "IL"]`) and/or `city` (string).

**Industry** — `naicsCodes` (array of strings). Pass full 6-digit codes (e.g. `"541512"`); the API filters on the first 2 digits internally.

**Size** — `employeeCountMin` / `employeeCountMax` (integers). `revenueMin` / `revenueMax` (integers in dollars, e.g. `10000000` for $10M).

**Incumbent broker** — `commercialBrokerName` or `benefitsBrokerName` (partial string match). Pass competitor names to find displacement targets: `"Marsh"`, `"AON"`, `"M3"`.

**Renewal timing** — `renewalMonths` (array of integers 1–12). Highest-conversion filter when combined with `states`.

**Compliance flags** — `hasDotViolations`, `hasOshaViolations`, `fidelityBondOutOfCompliance` (booleans).

**Data quality** — `minQualityScore` (integer, 0–100, default 51). Raise to 75+ for higher-confidence records; lower if results are too sparse. All previous diagnostics returned `qualityScore: 75`.

**Contact availability** — `hasContactEmails: true` restricts results to companies with email data available — use before running enriched contact lookups to reduce wasted calls.

**Pagination** — `pageToken` from `NextPageToken` in the response. `pageSize` max is 25.

### ⚠️ Broker field null handling

Broker fields (`leadCommercialBroker`, `leadBenefitsBroker`) are sourced from filing data (e.g. Form 5500). A null value means the data is **unavailable** — not that the company is unrepresented. **Never tell the user a company has no broker based solely on a null broker field.**

### Response fields to surface

- `leadCommercialBroker.name` — incumbent to displace
- `leadCommercialBroker.validated_date` — recency of broker intel
- `revenueRange` — size tier for prioritization
- `qualityScore` — data confidence; surface when ranking targets
- `isOutOfBusiness` — filter these out silently before presenting results
- `fidelityBonds` — compliance posture; `is_compliant: false` = compliance-motivated conversation opener
- `latitude` / `longitude` — available for proximity follow-up queries

### Typical prospecting workflow

1. Call `discovery_company_search` with `lineOfBusiness` + geography + NAICS + size filters
2. Filter out `isOutOfBusiness: true` results silently
3. Sort by `revenueRange` and `leadCommercialBroker` to identify best displacement targets
4. For top targets, call `discovery_company_contact_get` with `msid` and `enrich: false` to preview contacts
5. Confirm with user before calling with `enrich: true` — enrichment is billed per company

---

## discovery_company_contact_get

### ⚠️ Cost-gated — always preview before enriching

**`enrich: false` (default, no charge)** — Returns contact names, titles, and availability flags (`hasEmail`, `hasDirectPhone`, `hasMobilePhone`). Always start here.

**`enrich: true` (billed per company)** — Returns actual emails and phone numbers. Incurs a charge billed to the agency. **Always confirm with the user before setting `enrich: true`.** Never enrich silently.

### Lookup methods (priority order)

1. `msid` — fastest; pass directly from `discovery_company_search` results
2. `ein` + `name`
3. `name` + `city` + `state`

```
// Preview first — no charge
discovery_company_contact_get({ msid: "M84000062336619", enrich: false })

// Only after user confirms:
discovery_company_contact_get({ msid: "M84000062336619", enrich: true })
```

Use `pageSize` (max 25) and `pageToken` for companies with many contacts.

---

## research_brief_generate / research_brief_get

Generates a comprehensive AI research brief for a prospect. Requires `msid` and `lob` (`"Commercial"`, `"Benefits"`, or `"PersonalLines"`).

- Runs **synchronously** — do not call `research_brief_get` during generation
- Progress notifications include a `publicId` — save this for later retrieval
- `research_brief_get` is only for briefs from **prior sessions** using a saved `publicId`

---

## Tips

- Always filter `isOutOfBusiness: true` results before presenting — do this silently
- Default `minQualityScore` is 51; use 75 for the highest-confidence prospect lists
- Displacement play: filter by `commercialBrokerName` with a competitor, sort by `revenueRange.max` descending
- `hasContactEmails: true` in search filters to companies where enrichment will return email data — use to improve yield before spending credits
- Null broker fields mean unknown, not unrepresented — never imply a company is broker-free based on null
