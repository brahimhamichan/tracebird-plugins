---
name: getting-started
description: "First-run setup for the Tracebird plugin: verifies the connection, explains what Tracebird can do, checks which integrations are connected and guides the user to connect the missing ones. Use when the user says \"set up tracebird\", \"connect tracebird\", \"is tracebird working\", \"what can tracebird do\" or gets sign-in/unauthorized errors."
---
# Getting started with Tracebird

Tracebird (https://tracebird.io) is a Facebook and Google Ads manager with attributed revenue, front-end/back-end funnel revenue, conversion tracking, automation rules, split tests and reports. The Tracebird MCP server exposes every app operation as a tool.

## 1. Verify the connection
1. Call `get_workspace`. It returns the workspace, currency, timezone, plan, acting user and granted scopes.
2. If it succeeds, tell the user the workspace name, currency, timezone and whether the connection can write (write scope) or is read-only.
3. If the tools are missing or return unauthorized, the user must sign in:
   - Claude Code: run `/mcp`, pick tracebird, choose Authenticate.
   - Codex: run `codex mcp login tracebird` in a terminal.
   - Grok Build: open `/mcps` and sign in to tracebird.
   - The sign-in opens the browser; the user picks the workspace and should leave "write access" checked if they want changes made.
4. A write tool that returns a scope error means the connection is read-only: reconnect with write access.

## 2. Explain what is possible
Summarize in a few lines, with the matching skill:
- Performance and revenue analysis: `get_performance_summary`, `get_insights`, `get_revenue` (skill performance-review).
- Stopping wasted spend: pausing ads, ad sets, campaigns, shifting budget (cut-wasted-spend).
- Automation rules that pause, scale or alert on a schedule (build-automation).
- Landing-page split tests and custom link domains (split-testing).
- Pixel, conversion sources, CAPI, attribution and FE/BE revenue settings (tracking-setup).
- Reports, dashboards, embeds and API sync (reports-and-dashboards).
- Workspace, team, notifications, logs, webhooks, billing (workspace-admin).

## 3. Check integrations
1. Call `list_integrations`. Group the result as connected, errored/paused and not connected.
2. Call `list_ad_accounts` to confirm which Facebook and Google ad accounts are enabled.
3. For each important gap, recommend the next step:
   - Facebook, Google Ads, Google Analytics, Stripe need a browser sign-in: call `get_integration_connect_url` and give the returned connect_url to the user to open while signed in to Tracebird. Afterwards use `list_ad_account_choices` and `select_ad_accounts` to pick accounts.
   - CRMs and payments (GoHighLevel, HubSpot, Whop OAuth): `get_conversion_source_connect_url`.
   - Supabase or BigQuery sign-in: `get_supabase_connect_url`, `get_bigquery_connect_url`.
   - Calendly, Cortana, Whop API key, HYROS: ask the user for the key, then use the dedicated connect tool (see tracking-setup).
4. After a connection, `sync_integration` runs a manual sync; check `list_notifications` or `list_sync_history` if it fails.

## 4. Safety rules (apply to every Tracebird task)
- Read before write. Before any change to live ads, budgets, automations, tracking, integrations or settings, state the exact change (what, which ids, before to after, expected impact) and wait for explicit confirmation.
- Prefer reversible steps: pause instead of delete; create automation rules OFF and preview first.
- Dates are YYYY-MM-DD in the workspace reporting timezone. Money is in major units with a currency field.
- List tools are paginated: pass next_cursor back as cursor. On HTTP 429 wait for Retry-After before retrying.
- If a tool named in these skills is not in your tool list (Codex loads a compact list), find it with `search_tools` and run it with `call_read_tool` (read scope) or `call_tool` (write scope).

## 5. Finish
Offer a concrete first task (for example a 7-day performance review) and note what data is missing or stale.
