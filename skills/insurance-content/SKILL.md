---
name: insurance-content
description: >
  Search, retrieve, and deliver Zywave's library of insurance sales and
  compliance content. Find articles, guides, checklists, and risk management
  materials to share with clients or use in proposals. Use when the user asks
  to find content, sales materials, client-facing documents, or prospecting
  collateral for a specific industry, coverage line, or topic.
allowed-tools: [mcp__zywave__content_search, mcp__zywave__content_prospecting_search, mcp__zywave__get_content_full_text, mcp__zywave__download_content]
---

# Content with Zywave

## Tool selection guide

| Goal | Tool |
|---|---|
| General content search (any topic) | `content_search` |
| Industry + LOB specific prospecting content | `content_prospecting_search` |
| Read the full text of a content item | `get_content_full_text` |
| Get a download URL for a content file | `download_content` |

---

## content_search

Use for any natural-language content query not scoped to a specific industry. Results are ranked by semantic similarity score.

- `query`: Natural language, e.g. `"ransomware prevention checklist for small business"` or `"EPLI coverage explanation for HR managers"`
- `pageIndex`: Zero-based. Start at `0`, increment for more results
- `totalCount` in the response tells you how many items match — inform the user if there are many pages

**Result fields to surface:**
- `title` + `description` — show these as the primary summary
- `tagIdsByTagNames` — tag name keys describe content type and LOB
- `contentId` — required for `get_content_full_text` and `download_content` (integer)
- `fileDownloadUrl` — direct presigned URL to `content.zywave.com`; valid for a limited time

**Deduplication:** The same `contentId` may appear multiple times across pages with slightly different scores. Deduplicate by `contentId` before presenting results to the user.

**Note on resourceUri:** Results include a `resourceUri` in the format `zywave://content/download/{contentId}`. This is an internal reference only — use `fileDownloadUrl` or `download_content` for actual file access.

---

## content_prospecting_search

Use when the user wants content targeted at a specific industry and line of business — typically for prospect outreach or pitch preparation.

Required fields:
- `lob`: Must be exactly `"Commercial"`, `"Benefits"`, or `"PersonalLines"`
- `naicsCode`: 6-digit NAICS code string (e.g. `"541512"`). Filtered on first 2 digits server-side
- `query`: Natural language search term
- `pageIndex`: Zero-based
- `state`: `null` for national content, or a US state abbreviation/name for state-specific materials

`fileDownloadUrl` is populated directly in prospecting search results — no need to call `download_content` separately unless you need a fresh URL.

**Typical workflow for a prospect pitch:**
1. Get prospect NAICS from prior `discovery_companies_search` results
2. Call `content_prospecting_search` with that NAICS + `lob: "Commercial"` + relevant query
3. Present top 3–5 results by title/description
4. Offer full text or download link for items the user selects

---

## get_content_full_text

Returns stripped plain text of a content item. Use when the user wants to read, summarize, or quote from content within the conversation.

- `contentId` must be an integer — cast from string if needed
- HTML, scripts, and styles are stripped server-side; response is clean prose
- Summarize long documents unless the user explicitly asks for the full text

---

## download_content

Returns `{ fileName, downloadUrl }` where `downloadUrl` points to `content.zywave.com`. URLs are presigned and time-limited — generate on demand.

If `download_content` errors, fall back to `fileDownloadUrl` from the original search result (same domain, also presigned).

---

## Content tag reference

| Tag | Meaning |
|---|---|
| `Content On Command` | Broker-customizable template |
| `MLG Enabled` | Supports multi-language generation |
| `Zywave Content` | Polished, ready-to-share |
| `IRMI_Api` | Sourced from IRMI reference library |
| `Updated` | Recently revised |
| `Word Document` | Downloadable .doc/.docx |
| `HTML` | Web-rendered content |
| `Coverage Insights` | Coverage explanation series |
| `Insurance Considerations` | Industry-specific exposure guide |
| `Risk Management` | Risk-focused material |

For client-facing materials, prefer `Zywave Content` + `Article` or `Word Document` tagged items. For compliance-driven clients, look for `Risk Management`, `OSHA`, or `DOT` tags.
