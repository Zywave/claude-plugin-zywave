---
name: insurance-content
description: >
  Search, retrieve, and deliver Zywave's library of insurance sales and
  compliance content. Find articles, guides, checklists, and risk management
  materials to share with clients or use in proposals. Use when the user asks
  to find content, sales materials, client-facing documents, or prospecting
  collateral for a specific industry, coverage line, or topic.
allowed-tools: [mcp__zywave__content_search, mcp__zywave__content_prospecting_search, mcp__zywave__content_get_full_text, mcp__zywave__content_download]
---

# Content with Zywave

## Tool selection guide

| Goal | Tool |
|---|---|
| General content search (any topic) | `content_search` |
| Industry + LOB specific prospecting content | `content_prospecting_search` |
| Read the full text of a content item | `content_get_full_text` |
| Get a download URL for a content file | `content_download` |

---

## content_search

Use for any natural-language content query not scoped to a specific industry. Results are ranked by semantic similarity score.

- `query`: Describe a topic, risk, or client situation — not a single keyword. E.g. `"ransomware prevention checklist for small business"` or `"EPLI coverage explanation for HR managers"`. Vague single-word queries return poor results.
- `skip`: Zero-based offset for pagination. Start at `0`.
- `top`: Records per page, max 25. Defaults to 10.

**Result fields to surface:**
- `title` + `description` — primary summary to show the user
- `tagIdsByTagNames` — tag name keys describe content type and LOB
- `contentId` — required for `content_get_full_text` and `content_download` (integer)
- `fileDownloadUrl` — direct presigned URL to `content.zywave.com`; valid for a limited time

**Deduplication:** The same `contentId` may appear multiple times in results. Deduplicate by `contentId` before presenting to the user.

**Note on resourceUri:** Results include a `resourceUri` in the format `zywave://content/download/{contentId}`. This is an internal reference — always use `fileDownloadUrl` or `content_download` for actual file access, not `resourceUri`.

---

## content_prospecting_search

Use when the user wants content targeted at a specific industry and line of business — for prospect outreach or pitch prep.

Required fields:
- `lob`: Must be exactly `"Commercial"`, `"Benefits"`, or `"PersonalLines"`
- `naicsCode`: 6-digit NAICS code string (e.g. `"541512"`). Filtered on first 2 digits server-side
- `query`: Natural language search term
- `pageIndex`: Zero-based
- `state`: `null` for national content, or a US state abbreviation/name for state-specific materials

`fileDownloadUrl` is populated directly in results — no need to call `content_download` separately unless you need a fresh URL.

**Typical workflow for a prospect pitch:**
1. Get prospect NAICS from prior `discovery_company_search` results
2. Call `content_prospecting_search` with that NAICS + `lob: "Commercial"` + relevant query
3. Present top 3–5 results by title/description
4. Offer full text or download link for items the user selects

---

## content_get_full_text

Returns stripped plain text of a content item. Use when the user wants to read, summarize, or quote from content within the conversation.

- `contentId` must be an integer — cast from string if needed
- HTML, scripts, and styles are stripped server-side
- Summarize long documents unless the user explicitly asks for the full text dump

---

## content_download

Returns `{ fileName, downloadUrl }` where `downloadUrl` points to `content.zywave.com`. URLs are presigned and time-limited — generate on demand, not in advance.

If `content_download` errors, fall back to `fileDownloadUrl` from the original search result — same domain, also presigned.
