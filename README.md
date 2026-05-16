# Zywave Plugin for Claude Code

Connect Claude Code to Zywave's platform. Prospect for new accounts, manage your book of business, and surface client-ready content — all through natural language in your terminal.

---

## What this plugin does

**Business Prospecting** — Search Zywave's market intelligence database to find commercial P&C and employee benefits prospects by industry, geography, size, incumbent broker, renewal month, or compliance posture. Pull decision-maker contacts for identified targets. 

**CRM** — Read and manage your Zywave book of business. Search, create, update, and archive commercial and personal lines accounts and contacts.

**Content** — Search Zywave's library of insurance articles, guides, checklists, and risk management materials. Retrieve full text or download links for client-facing content and prospect pitch materials.

---

## Requirements

- A Zywave account with MCP API access enabled
- Claude Code v2.x or later

This plugin accesses **your** Zywave data. Authentication is tied to your Zywave identity

---

## Installation

```bash
/plugin install zywave@claude-plugins-official
```

After installation, authenticate:

```bash
/plugin zywave auth
```

This opens a browser window to complete OAuth login with your Zywave credentials. The session token is stored locally and refreshed automatically.

---

## Authentication details

The plugin uses **OAuth 2.0 with PKCE** via Zywave's identity provider (`auth.zywave.com`).

**Scopes requested:**
- `api.accounts` — read/write your CRM accounts and contacts
- `api.content` — search and download content library items
- `api.companies` — search Zywave's external market intelligence database
- `offline_access` — enables token refresh without re-authentication

Tokens are stored in your local Claude Code credential store and are never transmitted outside of calls to `ai.zywave.com`. Sessions expire after 1 hour of inactivity; the plugin refreshes automatically if `offline_access` is granted.

**If authentication fails:** Verify your Zywave account has MCP API access enabled. Contact your Zywave account manager or email support@zywave.com.

---

## Available tools

### Prospecting
| Tool | Description |
|---|---|
| `companies_commercial_search` | Find P&C prospects by broker, NAICS, geography, size, renewal month |
| `companies_benefits_search` | Find employee benefits prospects with same filter set |
| `company_contacts_get` | Get decision-maker contacts for a prospect company |

### CRM
| Tool | Description |
|---|---|
| `account_search` | Search your book of business with OData filters |
| `account_get` | Get full details on a single account |
| `account_create` | Create a new account (with duplicate detection) |
| `account_update` | Update account fields |
| `account_delete` | Archive (recoverable) or permanently delete an account |
| `account_restore` | Restore an archived account |
| `account_contact_search` | Search contacts across your accounts |
| `account_contact_get` | Get a single contact's details |
| `account_contact_create` | Add a contact to an account |
| `account_contact_update` | Update contact fields |
| `account_contact_delete` | Archive or permanently delete a contact |
| `account_contact_restore` | Restore an archived contact |
| `account_get_supported_lines_of_business` | List valid LOB values for create/update |
| `household_contacts_get` | Get personal lines household contacts |

### Content
| Tool | Description |
|---|---|
| `content_search` | Semantic search across Zywave's full content library |
| `content_prospecting_search` | Industry + LOB scoped content for prospect outreach |
| `get_content_full_text` | Read the full text of a content item |
| `download_content` | Get a presigned download URL for a content file |

### System
| Tool | Description |
|---|---|
| `system_who_am_i` | Returns your authenticated identity and session details |

---

## Example prompts

**Prospecting:**
```
Find technology companies in Wisconsin with 50–500 employees currently with M3 Insurance
```
```
Show me manufacturing prospects renewing in Q1 with any OSHA violations
```
```
Get the decision-maker contacts at Concurrency Inc in Brookfield WI
```

**CRM:**
```
Show me all my active commercial accounts
```
```
Create a new account for Acme Corp, commercial, NAICS 541512, based in Milwaukee WI
```
```
Archive the account for KK Personal Lines 73
```

**Content:**
```
Find cyber liability articles I can send to a technology firm prospect
```
```
Get the full text of the Ransomware Survival Guide
```
```
Find prospecting content for a manufacturing company (NAICS 332999) in Wisconsin
```

---

## Known limitations

- **Content search** requires an active Zywave content subscription. If `content_search` returns an error, verify your subscription includes Content on Command.
- **Download URLs** from `download_content` are presigned and time-limited (~15 minutes). Generate them on demand.
- **Permanent deletion** via `account_delete` with `permanent: true` is irreversible. Claude Code will always confirm before executing.
- `account_get_supported_lines_of_business` must be called before setting `linesOfBusiness` on create/update — valid values are tenant-specific.

---

## Data and privacy

All API calls go directly to `ai.zywave.com` over HTTPS. No data is stored by the plugin beyond your local OAuth token. Zywave's data handling is governed by your existing service agreement with Zywave. See [Zywave's Privacy Policy](https://www.zywave.com/privacy-statement/) for details.

---

## Support

- **Zywave support:** support@zywave.com
- **Plugin issues:** https://github.com/zywave/claude-plugin-zywave/issues
- **Zywave developer docs:** https://booster.zywave.dev/

---

## License

Proprietary — use subject to your Zywave service agreement. See [Zywave Terms of Service](https://www.zywave.com/terms-conditions/).
