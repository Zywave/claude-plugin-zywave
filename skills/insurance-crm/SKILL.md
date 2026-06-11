---
name: insurance-crm
description: >
  Manage the user's Zywave book of business: search, view, create, update,
  and archive accounts and contacts. Use when the user refers to "my clients",
  "my accounts", "my book", or asks to look up, add, or update a specific
  client or contact they already manage. Do NOT use for market prospecting —
  use insurance-prospecting for that.
allowed-tools: [mcp__zywave__account_search, mcp__zywave__account_get, mcp__zywave__account_create, mcp__zywave__account_update, mcp__zywave__account_delete, mcp__zywave__account_restore, mcp__zywave__account_contact_search, mcp__zywave__account_contact_get, mcp__zywave__account_contact_create, mcp__zywave__account_contact_update, mcp__zywave__account_contact_delete, mcp__zywave__account_contact_restore, mcp__zywave__system_who_am_i]
---

# CRM with Zywave

## Tool selection guide

| Task | Tool |
|---|---|
| Search for existing accounts | `account_search` |
| Get full details on one account | `account_get` |
| Create a new account | `account_create` |
| Update an existing account | `account_update` |
| Archive (soft-delete) an account | `account_delete` (permanent: false) |
| Permanently delete an account | `account_delete` (permanent: true) — **confirm with user first** |
| Restore an archived account | `account_restore` |
| Search contacts across accounts | `account_contact_search` |
| Get one contact's details | `account_contact_get` |
| Add a contact to an account | `account_contact_create` |
| Update a contact | `account_contact_update` |
| Archive/delete a contact | `account_contact_delete` |
| Restore an archived contact | `account_contact_restore` |
| Verify current session identity | `system_who_am_i` |

---

## account_search — OData filter patterns

`linesOfBusiness` is **not filterable** via OData — retrieve results and inspect the field client-side.

`clientSize` filterable values: `"From0To25"`, `"From26To50"`, `"From51To99"`, `"From100To499"`, `"From500To999"`, `"From1000To2499"`, `"From2500To4999"`, `"MoreThan4999"`

Common filter patterns:

```
// Active commercial accounts only
"isArchived eq false and classification eq 'Commercial'"

// Search by name (contains)
"contains(name, 'Acme')"

// Filter by client size tier
"clientSize eq 'From100To499' and isArchived eq false"

// Recently updated (ISO 8601)
"updatedDateTime gt 2025-01-01T00:00:00Z"

// Combined
"classification eq 'Commercial' and isArchived eq false and contains(name, 'Tech')"
```

Pagination: `skip` (zero-based offset) + `top` (max 100). Check `hasMoreResults` and `totalCount` to determine whether to paginate.

---

## account_create — duplicate handling

The API returns potential duplicates before creating. **Always surface these to the user** and ask for confirmation before retrying with `forceCreate: true`. Never set `forceCreate: true` without explicit user acknowledgment.

### Valid linesOfBusiness values

`"Benefits Client"`, `"Benefits Prospect"`, `"Commercial Lines Client"`, `"Commercial Lines Prospect"`, `"P&C Client"`, `"P&C Prospect"`, `"Personal Lines Client"`, `"Personal Lines Prospect"`

---

## Destructive operations — safety rules

**Archiving** (`account_delete` with `permanent: false`): recoverable. Confirm intent but proceed without extended warnings.

**Permanent deletion** (`account_delete` with `permanent: true`): **irreversible**. Always state clearly: _"This permanently deletes the account and cannot be undone."_ Require explicit confirmation before proceeding. Never infer confirmation from ambiguous phrasing.

Same rules apply to `account_contact_delete`.

---

## account_contact_search — useful filter patterns

```
// All contacts for a specific account
"accountId eq 12345678"

// Primary contacts only
"isPrimaryContact eq true and isArchived eq false"

// Find by email
"emailAddress eq 'john.doe@example.com'"

// Active contacts across all accounts
"isActive eq true and isArchived eq false"
```

---

## account_update / account_contact_update

Both are **partial updates** — only pass fields that are changing. Omitted fields retain their current values.

`linesOfBusiness` on `account_update` **replaces all existing values** — pass the full intended array, not just additions. Pass `[]` to clear all LOBs.

Similarly, `naicsCodes` on `account_update` replaces all existing codes when provided.

---

## Tips

- `msid` on an account record links to external market data — pass it to `discovery_company_contact_get` to pull external contacts for that same company
- `clientSize` is filterable in `account_search` — use it to segment by business size tier
- `linesOfBusiness` is inspect-only in search results; filter client-side after retrieval
- Use `system_who_am_i` to confirm which org/agency context is active if the user reports unexpected results
