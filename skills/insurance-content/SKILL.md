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
- `pageIndex`: Zero-based page number. Start at `0`, increment for more results
- `totalCount` in the response tells you how many items match — inform the user if there are many pages

**Result fields to surface:**
- `title` + `description` — show these as the primary summary
- `tagIdsByTagNames` — use tag names (keys) to describe the content type and LOB to the user
- `contentId` — required for `get_content_full_text` and `download_content`
- `fileDownloadUrl` — direct presigned URL, valid for a limited time; share this if the user wants to download immediately

**Deduplication note:** The same `contentId` may appear multiple times in results with slightly different scores. Deduplicate by `contentId` before presenting results to the user.

---

## content_prospecting_search

Use when the user wants content targeted at a specific industry and line of business — typically for prospect outreach or pitch preparation.

Required fields:
- `lob`: Must be exactly `"Commercial"`, `"Benefits"`, or `"PersonalLines"` — no other values accepted
- `naicsCode`: 6-digit NAICS code string (e.g., `"541512"`). Results are filtered by the first 2 digits, so passing a full code is fine
- `query`: Natural language search term
- `pageIndex`: Zero-based
- `state`: Can be `null` for national content, or a US state name/abbreviation to filter to state-specific materials

**Typical usage pattern for a prospect pitch:**
1. Identify prospect's NAICS code from prior `companies_commercial_search` results
2. Call `content_prospecting_search` with that NAICS + `lob: "Commercial"` + a query like `"risk exposures coverage gaps"`
3. Present the top 3–5 results by title/description
4. Offer to pull full text or download links for items the user wants

---

## get_content_full_text

Returns stripped plain text of a content item. Use this when the user wants to read, summarize, or quote from a piece of content within the conversation.

- Pass `contentId` from search results (integer, not string — cast if needed)
- HTML, scripts, and styles are stripped server-side; the response is clean prose
- For long documents, summarize rather than dumping the full text unless the user explicitly requests it

---

## download_content

Returns a presigned download URL for a content file.

- Pass `contentId` from search results
- URLs are time-limited — generate them on demand, not in advance
- The `fileDownloadUrl` field in search results is also a valid direct link; no need to call `download_content` separately unless you need a fresh URL

**Note:** If `download_content` returns an error, fall back to `fileDownloadUrl` from the original search result.

---

## Tips

- For client-facing materials, prefer content tagged `"Zywave Content"` and `"Article"` or `"Guide / Plan"` — these are polished and ready to share
- For compliance-driven clients, filter by tags like `"Risk Management"`, `"Coverage / Insurance"`, `"OSHA"`, or `"DOT"`
- When building a prospect pitch deck, combine `content_prospecting_search` (for industry-specific risk articles) with `companies_commercial_search` (for prospect intel) for a complete picture
