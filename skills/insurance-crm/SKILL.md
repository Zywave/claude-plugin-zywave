---
name: insurance-crm
description: >
  Manage the user's Zywave book of business: search, view, create, update,
  and archive accounts and contacts. Use when the user refers to "my clients",
  "my accounts", "my book", or asks to look up, add, or update a specific
  client or contact they already manage. Do NOT use for market prospecting —
  use insurance-prospecting for that.
allowed-tools: [mcp__zywave__account_search, mcp__zywave__account_get, mcp__zywave__account_create, mcp__zywave__account_update, mcp__zywave__account_delete, mcp__zywave__account_restore, mcp__zywave__account_contact_search, mcp__zywave__account_contact_get, mcp__zywave__account_contact_create, mcp__zywave__account_contact_update, mcp__zywave__account_contact_delete, mcp__zywave__account_contact_restore, mcp__zywave__account_get_supported_lines_of_business, mcp__zywave__household_contacts_get]
---

# Insurance CRM with Zywave

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
| Get personal lines household contacts | `household_contacts_get` |
| Get valid LOB values for create/update | `account_get_supported_lines_of_business` |

---

## account_search — OData filter patterns

The `filter` field is required. Use OData `$filter` syntax. Common patterns:

```
// Active commercial accounts only
"isArchived eq false and classification eq 'Commercial'"

// Search by name (contains)
"contains(name, 'Acme')"

// Accounts in a specific state
"state eq 'WI' and isArchived eq false"

// Recently updated (ISO 8601)
"updatedDateTime gt 2025-01-01T00:00:00Z"

// Combined
"classification eq 'Commercial' and isArchived eq false and contains(name, 'Tech')"
```

Pagination: `skip` (zero-based offset) + `top` (max 100). Check `hasMoreResults` and `totalCount` to determine whether to paginate.

---

## account_create — duplicate handling

The API returns a list of potential duplicates before creating. **Always surface these to the user** and ask for confirmation before retrying with `forceCreate: true`. Never set `forceCreate: true` without explicit user acknowledgment.

Before calling `account_create` with `linesOfBusiness`, call `account_get_supported_lines_of_business` to get valid enum values — do not guess LOB strings.

---

## Destructive operations — safety rules

**Archiving** (`account_delete` with `permanent: false`): recoverable. Confirm intent but proceed without extended warnings.

**Permanent deletion** (`account_delete` with `permanent: true`): **irreversible**. Always state clearly: _"This permanently deletes the account and cannot be undone."_ Require explicit confirmation ("yes, permanently delete") before proceeding. Never infer confirmation from ambiguous phrasing.

Same rules apply to `account_contact_delete`.

---

## contact_search — useful filter patterns

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

## account_update

Only pass fields that are changing — the API does partial updates. Do not re-send all existing field values. Retrieve the account first with `account_get` if you need to verify current state before updating.

---

## Tips

- `msid` in account records links to external market data — use it as input to `company_contacts_get` in the prospecting skill to pull external contacts for the same company
- `linesOfBusiness` is a relationship array (e.g., `["Commercial Client", "Benefits Prospect"]`). Always validate values with `account_get_supported_lines_of_business` before setting them
- If the user asks about "household" contacts, those are personal lines — use `household_contacts_get` with the account's `msid`
