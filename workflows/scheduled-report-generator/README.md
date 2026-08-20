# Scheduled Report Generator

## Problem it solves

Key metrics (signups, conversions, etc.) usually live in a database that nobody checks unless someone remembers to run a query. This workflow removes that manual step: every day at a fixed time it queries the database itself, summarizes new activity with a day-over-day comparison, and emails a readable report to whoever needs visibility — no dashboard login or manual SQL required. If the query fails for any reason, the workflow still sends an email saying so, instead of the report just silently not showing up.

## Flow diagram

```
Schedule Trigger (daily, cron-configurable — default 08:00)
        |
        v
Query New Signups (Last 24h)   (Postgres, placeholder connection —
                                 per-plan today vs. yesterday counts,
                                 continues on failure instead of stopping the run)
        |
        v
Query Failed?   (IF)
   |                          \
   | true (query errored)      | false (query succeeded)
   v                            v
Build Failure Email       Aggregate & Summarize Results
(fallback message,               |
 includes the error)             v
   |                        Format HTML Report
   |                              |
   +--------------+---------------+
                  v
          Send Report Email
```

## Nodes used

| Node | Purpose |
|---|---|
| Schedule Trigger - Daily 08:00 | Trigger. Cron expression `0 8 * * *` by default — edit the node to change the time/frequency |
| Query New Signups (Last 24h) | Postgres node running a single query that returns per-plan counts for the last 24h *and* the 24h before that (via `FILTER`), so today-vs-yesterday comparison doesn't need a second query. `continueOnFail` is enabled so a DB outage doesn't kill the run |
| Query Failed? | IF node — branches on whether the query errored |
| Build Failure Email | Set node — composes a fallback `subject`/`htmlBody` explaining the report couldn't be generated, including the underlying error |
| Aggregate & Summarize Results | Code node — computes total signups today, total yesterday, percent change, and a per-plan breakdown sorted by volume |
| Format HTML Report | Code node — renders the summary as a simple HTML table and composes the real `subject`/`htmlBody` |
| Send Report Email | Email node (placeholder SMTP credentials) — sends whichever `subject`/`htmlBody` reached it (success or failure path) to the configured recipient list |

## How to import into n8n

1. In n8n, go to **Workflows → Add Workflow → Import from File** (or **⋮ → Import from File** on the workflows list).
2. Select [workflow.json](workflow.json) from this folder.
3. The workflow will appear with all 7 nodes and connections wired up, but **no credentials attached** — see setup below before activating it.

## Configuring the schedule, DB connection, and SMTP credentials

1. **Schedule** — open the **Schedule Trigger - Daily 08:00** node and adjust the cron expression (`0 8 * * *` = 08:00 server time, daily) to whatever time/frequency you want.
2. **Database** — open the **Query New Signups (Last 24h)** node, create/select a Postgres credential in n8n pointing at your real database, and update the query's table/column names (`subscriptions`, `plan`, `created_at`) to match your schema. The credential itself is configured in n8n's Credentials UI and is only referenced by name/id in the workflow — never stored in `workflow.json`.
3. **SMTP** — open the **Send Report Email** node and create/select an SMTP credential in n8n for your mail provider. Update the `fromEmail` field to a real sending address.
4. **Recipients** — set `REPORT_RECIPIENT_EMAILS` (see below) to a comma-separated list of report recipients.

## Credentials & environment variables needed

This workflow ships with no real secrets — placeholder credential references only. It reads recipient addresses from an environment variable at execution time (see the root [.env.example](../../.env.example)):

| Variable | Used by | Purpose |
|---|---|---|
| `REPORT_DB_HOST` | Postgres credential (configured in n8n, not in the workflow file) | Database host |
| `REPORT_DB_NAME` | Postgres credential | Database name |
| `REPORT_DB_USER` | Postgres credential | Database user |
| `REPORT_DB_PASSWORD` | Postgres credential | Database password |
| `REPORT_SMTP_HOST` | SMTP credential (configured in n8n) | Mail server host |
| `REPORT_SMTP_USER` | SMTP credential | Mail server username |
| `REPORT_SMTP_PASSWORD` | SMTP credential | Mail server password |
| `REPORT_RECIPIENT_EMAILS` | Send Report Email node | Comma-separated list of report recipients, read directly via `$env.REPORT_RECIPIENT_EMAILS` |

The `REPORT_DB_*` and `REPORT_SMTP_*` variables aren't read directly by node expressions — they document what values to use when creating the Postgres and SMTP credentials in n8n's Credentials UI, so the connection details stay out of version control entirely.

## Failure fallback behavior

If the **Query New Signups (Last 24h)** node fails (bad connection, timeout, schema change, etc.), the workflow does not fail silently. `continueOnFail` lets execution continue past the error, the **Query Failed?** IF node detects it, and **Build Failure Email** composes a clear "report generation failed" email — including the underlying error message — which is sent through the same **Send Report Email** node used for a successful report. Recipients always get *something* in their inbox at report time, even when the data pull breaks.
