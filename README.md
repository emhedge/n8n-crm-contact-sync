# n8n CRM Contact Sync

A scheduled n8n workflow that detects new and updated contacts in a source CRM, compares them against a target CRM, and syncs only the delta — appending new contacts and updating changed ones in place.

Built as a portfolio project demonstrating change detection, bidirectional data comparison, and conditional branching logic — patterns directly applicable to ERP/CRM migration and integration work.

---

## What it does

1. **Reads** all contacts from the source CRM (Google Sheets)
2. **Reads** all contacts from the target CRM (Google Sheets)
3. **Compares** both datasets to detect new contacts and updated contacts (via `last_updated` field)
4. **Forks** into two paths:
   - New contacts → appended to target
   - Updated contacts → existing rows updated in place
5. **Skips** gracefully when nothing has changed

---

## Architecture

```
Schedule Trigger (every 15 min)
    │
    ▼
Read CRM Source (Google Sheets)
    │
    ▼
Read CRM Target (Google Sheets)
    │
    ▼
Detect New & Updated Contacts (Code)
    ├── Extract New Contacts (Code)
    │       │
    │       ▼
    │   Has New Contacts? (IF)
    │       │
    │       └── true → Append New Contacts (Google Sheets)
    │
    └── Extract Updated Contacts (Code)
            │
            ▼
        Has Updated Contacts? (IF)
            │
            └── true → Update Existing Contacts (Google Sheets)
```

---

## Tech stack

| Tool | Purpose |
|---|---|
| [n8n](https://n8n.io) | Workflow automation engine |
| [Google Sheets](https://sheets.google.com) | Source and target CRM (simulated) |
| Docker | n8n containerized deployment |

---

## Data schema

Both source and target sheets share the same schema:

| Field | Description |
|---|---|
| `contact_id` | Unique identifier — used as the matching key |
| `first_name` | Contact first name |
| `last_name` | Contact last name |
| `email` | Contact email address |
| `company` | Company name |
| `status` | Contact status (Active, Inactive, VIP, etc.) |
| `last_updated` | ISO date — change detection key |

---

## How change detection works

- **New contact:** `contact_id` exists in source but not in target → append
- **Updated contact:** `contact_id` exists in both, but `last_updated` differs → update in place
- **Unchanged contact:** `contact_id` and `last_updated` match → skipped

This pattern is directly applicable to production sync scenarios — e.g. syncing contacts from Salesforce into Odoo, or orders from WooCommerce into Cin7, where only net-new or modified records should be written to avoid unnecessary API calls and data overwrites.

---

## Prerequisites

- n8n running locally via Docker (port 5678)
- Google account with Sheets API and Drive API enabled
- Google Sheets credential configured in n8n

---

## Setup

### 1. Start n8n via Docker

```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -e N8N_HOST=localhost \
  -e N8N_PORT=5678 \
  -e N8N_PROTOCOL=http \
  -e WEBHOOK_URL=http://localhost:5678 \
  -v "/path/to/your/n8n/data:/home/node/.n8n" \
  n8nio/n8n:latest
```

### 2. Create Google Sheets

Create two Google Sheets with identical headers:

```
contact_id | first_name | last_name | email | company | status | last_updated
```

- **CRM Source** — populate with contacts
- **CRM Target** — leave empty initially

### 3. Import the workflow

1. Open n8n at `http://localhost:5678`
2. Workflows → Import → upload `crm-contact-sync.json`
3. Update Google Sheets credentials in each node
4. Update spreadsheet names to match your sheets

### 4. Activate the workflow

Toggle the workflow to **Active**. It will run every 15 minutes automatically.

---

## Testing

**Initial sync:**
1. Populate CRM Source with contacts
2. Leave CRM Target empty
3. Execute workflow — all contacts should appear in CRM Target

**Update detection:**
1. Change a contact's `status` and `last_updated` in CRM Source
2. Execute workflow — only that contact's row should update in CRM Target, no duplicates

**No changes:**
1. Execute workflow with no changes in CRM Source
2. Both paths should skip gracefully with no writes

---

## n8n skills demonstrated

- Schedule Trigger configuration
- Google Sheets read and write operations
- Multi-input Code node referencing upstream nodes by name
- Change detection logic (new vs updated vs unchanged)
- Conditional forking with IF nodes
- Append vs Update Row operations
- Graceful skip handling for empty result sets
- Always Output Data pattern for empty-sheet edge cases

---

## Related projects

- [n8n Lead Intake CRM Workflow](https://github.com/emhedge/n8n-Lead-Intake-CRM-Workflow) — webhook-triggered lead pipeline with validation, enrichment, and notifications
- [n8n WooCommerce Order Sync](https://github.com/emhedge/n8n-woocommerce-order-sync) — scheduled order sync from WooCommerce to Google Sheets

---

## Author

Evyn Hedgpeth
[linkedin.com/in/evynhedgpeth](https://linkedin.com/in/evynhedgpeth) · [medium.com/@emhedge](https://medium.com/@emhedge)
