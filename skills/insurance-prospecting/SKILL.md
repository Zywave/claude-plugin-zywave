---
name: insurance-prospecting
description: >
  Prospect for commercial insurance and employee benefits accounts using
  Zywave's market intelligence database. Find target companies by industry
  (NAICS), geography, employee count, revenue, incumbent broker, renewal
  month, or compliance posture. Retrieve decision-maker contacts for
  identified prospects. Use when the user wants to find new accounts to
  pursue, displace an incumbent broker, or build a target list.
allowed-tools: [mcp__zywave__companies_commercial_search, mcp__zywave__companies_benefits_search, mcp__zywave__company_contacts_get]
---

# Insurance Prospecting with Zywave

## When to use which tool

| Goal | Tool |
|---|---|
| Find P&C / commercial prospects | `companies_commercial_search` |
| Find employee benefits prospects | `companies_benefits_search` |
| Get decision-maker contacts for a prospect | `company_contacts_get` |

Do **not** use these tools to look up accounts the user already manages — use `account_search` from the insurance-crm skill instead.

---

## companies_commercial_search / companies_benefits_search

### Key filters and how to use them

**Geography** — use `states` (array) and/or `city`. Always use state abbreviations (`"WI"`, not `"Wisconsin"`).

**Industry** — use `naicsCodes` (array of strings). Always pass the full 6-digit NAICS code (e.g., `"541512"` for custom computer programming). You can pass multiple codes to broaden the search.

**Size** — `employeeCountMin` / `employeeCountMax` are integers. `revenueMin` / `revenueMax` are integers in dollars (e.g., `10000000` for $10M).

**Incumbent broker** — `commercialBrokerName` (or `benefitsBrokerName`) accepts a partial string. Use this to find accounts currently held by a competitor (e.g., `"Marsh"`, `"AON"`, `"M3"`).

**Renewal timing** — `renewalMonths` is an array of integers (1–12). Pass `[1, 2, 3]` to find accounts renewing in Q1.

**Compliance flags** — `hasDotViolations`, `hasOshaViolations`, `fidelityBondOutOfCompliance` are booleans. Use these to identify high-risk or compliance-motivated prospects.

### Output fields to highlight to the user

- `leadCommercialBroker.name` — incumbent broker to displace
- `leadCommercialBroker.validated_date` — how current the broker intel is
- `revenueRange` — size tier for prioritization
- `fidelityBonds` — compliance status; high `bond_ratio` may indicate over-bonded or non-compliant
- `msid` — use this as input to `company_contacts_get`

### Typical prospecting workflow

1. Run `companies_commercial_search` with geography + NAICS + size filters
2. Sort/filter results by `leadCommercialBroker` to identify displacement opportunities
3. Pick high-value targets and call `company_contacts_get` with the `msid` to get decision-maker contacts
4. Surface contact names, titles, and phone/email to the user for outreach

---

## company_contacts_get

Requires one of: `msid`, or `(ein + name)`, or `(ein + address)`, or `(name + city + state)`.

The fastest path after a prospect search is to pass the `msid` directly from search results. Do not ask the user for it — extract it from the prior search response.

```
// Good — use msid from prior search
company_contacts_get({ msid: "M84000062336619" })

// Also valid if msid not available
company_contacts_get({ name: "Concurrency Inc", city: "Brookfield", state: "WI" })
```

---

## Tips for better results

- Start broad (state + NAICS only), then narrow with additional filters based on result volume
- `totalCount` in the response tells you how many records match — if >50, add filters before showing results
- For renewal-based prospecting, combine `renewalMonths` with `states` — this is the highest-conversion filter combination
- When the user says "who does [broker] own in [market]", pass that broker name to `commercialBrokerName` — this is a displacement play
