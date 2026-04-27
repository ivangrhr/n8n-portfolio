# Invoice request emails + reminders (MV Accounting)

n8n workflow for a monthly accounting practice: ask active clients for invoices in Bulgarian or English, log each run in Google Sheets, optionally follow up with reminders, and notify the operator by email.

## The problem

Collecting client invoices every month is repetitive: different languages, tracking who was contacted, who replied, and who needs a nudge. Doing it manually across many clients does not scale.

## The solution

Two scheduled branches in one workflow:

1. **Initial request (1st of the month, 09:00)**  
   Reads active clients from a **clients** sheet, filters `active = true`, assigns a `run_id` in the form `YYYY-MM-initial`, routes by `language` (`BG` vs `EN`), builds subject/body, sends email via **Gmail**, appends a row to **send_log**, loops clients with **Split In Batches**, aggregates, and emails the operator that drafts were generated (wording assumes review of Gmail drafts in your setup).

2. **Reminder path (7th of the month, 09:00)**  
   Loads **send_log** rows for the current month’s initial run (`status = initial_draft_created`, matching `run_id`). For each client, checks **Gmail** (inbox messages this month from a configured sender) to infer whether they replied. If a message exists, updates the sheet (`reply_received`, `completed_client_replied`). If not, builds BG or EN reminder copy, merges, and (in the exported version) flows through **Gmail → Create draft** (currently disabled in the JSON) into a sheet update for reminder timestamps/status.

A separate branch reads the month’s **send_log**, runs a **Code** node to count replied / reminder / still-initial rows, and emails the operator a **reminder-run summary**.

## Architecture (high level)

| Area | Nodes |
|------|--------|
| Data | Google Sheets (clients + send_log) |
| Email | Gmail (send, search inbox, optional draft) |
| Control flow | Filter, Switch (language), IF (reply?), Split In Batches, Merge |
| Ops visibility | Aggregate + Gmail summary to the operator |

## Google Sheets expectations

Wire your own spreadsheet in n8n after import. The workflow expects roughly:

**Sheet `clients` (or equivalent):** columns such as `active`, `client_id`, `company_name`, `contact_name`, `email`, `language` (`BG` / `EN`), plus any other fields you keep.

**Sheet `send_log`:** `run_id`, `client_id`, `company_name`, `email`, `language`, `initial_draft_created_at`, `initial_sent_at`, `reply_received`, `reminder_draft_created_at`, `status` (e.g. `initial_draft_created`, `completed_client_replied`, `reminder_draft_created`).

## Import and credentials

1. In n8n: **Workflows → Import from File** and select `Имейл за искане на фактури + напомняне.json`.
2. Reconnect **Google Sheets OAuth2** and **Gmail OAuth2** credentials to your accounts.
3. Point every **Google Sheets** node to your document and the correct sheets/tabs.
4. Adjust **Schedule** triggers (cron) if you want different days or times.
5. Replace hard-coded operator addresses in Gmail “summary” nodes with your own.
6. Review the **Check reply** Gmail filter (`sender`, `receivedAfter`, labels): align `sender` with the mailbox or addresses you actually use to detect client replies.

## Notes on this export

- Sticky notes in the canvas mention a possible move to Outlook and HTML email; treat those as product backlog, not runtime requirements.
- **Create a draft** (reminder branch) is **disabled** in the exported workflow—enable and test before relying on reminder drafts.
- Credential IDs and a sample spreadsheet ID inside the JSON are from the author’s dev instance; they are not valid on your machine until you replace them.
- **Pinned data** on `Read clients` is example test data for development; clear or replace pin data for production use.

## Workflow file

- `Имейл за искане на фактури + напомняне.json` — full n8n workflow export in this folder (Bulgarian workflow title: email for invoice request + reminder).

## Tech stack

**Schedule Trigger**, **Google Sheets**, **Gmail**, **Filter**, **Switch**, **Set**, **Merge**, **Split In Batches**, **Aggregate**, **IF**, **Code** (JavaScript).
