---
name: reports-and-dashboards
description: "Builds Tracebird reports (metrics, templates, delivery schedules, saved views), customizes the dashboard layout, creates public embeds, and syncs Tracebird data into another app through the REST API. Use when the user says \"create a report\", \"schedule a weekly report\", \"change my dashboard\", \"embed the dashboard\", \"pull data into my app\", \"use the Tracebird API\"."
---
# Reports, dashboards and data export

## 1. Reports
1. `get_workspace`. `list_reports` first to avoid duplicates (archived are hidden unless asked).
2. Always call `list_report_metrics` before building: it lists valid metric keys, templates and the row limit. CRM metrics show data only if CRM conversions are recorded.
3. Clarify audience, metrics, rows, comparison (for example previous period) and timeframe.
4. `create_report` with name, description, template and config (metrics, rows, comparison). Show the definition back to the user.
5. Maintain: `get_report`, `update_report`, `duplicate_report`, `archive_report` (prefer archive to removal).

## 2. Schedules (saved delivery preferences)
1. `list_report_schedules`. Schedules are saved preferences, like the Reports page's "Saved schedule preferences"; don't tell the user an email was sent.
2. Confirm recipients (1-25), cadence (daily, weekly or monthly), timezone (IANA, default UTC) and format (pdf or csv) with the user, then `create_report_schedule`.
3. Change or pause with `update_report_schedule` (enabled false pauses).

## 3. Saved views
- Report pages: `list_report_views`, `create_report_view`, `update_report_view`.
- Ads Manager filters: `list_manager_views`, `create_manager_view` (name, filters, hidden_columns; shared with the workspace), `update_manager_view`, `delete_manager_view`.
- Personal table views: `get_table_views`, then `update_table_views` with the complete list (last write wins, so send everything you read plus changes).

## 4. Dashboard layout
1. `get_dashboard_preferences` for the current layout and updated_at.
2. `list_dashboard_options` for valid metric cards, campaign columns, chart palettes and limits.
3. `update_dashboard_preferences` with only the fields to change (cards, secondaryCards, chartSeries, campaignColumns, chartType, and so on). Pass expected_updated_at to avoid overwriting a concurrent edit. It affects only the acting user's layout.
4. Data for the dashboard: `get_dashboard_revenue`, `get_dashboard_trend`.

## 5. Public embeds (anyone with the link sees the numbers)
1. `get_dashboard_embed` shows the current link or none.
2. Confirm what will be public, then `create_dashboard_embed` (creating again rotates the link and breaks the old URL).
3. Display settings: `list_embed_account_options`, `update_dashboard_embed_config`, `preview_dashboard_embed` and `preview_dashboard_embed_widget`.
4. Protect it: `set_dashboard_embed_password`, `clear_dashboard_embed_password`; disable with `revoke_dashboard_embed`.

## 6. Syncing data to another app (REST API)
Same operations as the MCP tools. Tell the user:
- Base URL https://tracebird.io/api/v1, header Authorization: Bearer <API key>. Keys are created in Settings, API and MCP (the key cannot be created via tools; `list_api_keys` shows metadata only). Use a read-only key for exports.
- Responses are lists with data, has_more and next_cursor; follow next_cursor as the cursor query parameter. Errors return error.code and message.
- Incremental sync: campaigns, ad sets and ads are ordered by updated_at; store the newest updated_at and pass it as updated_since next run. Conversions: pass the newest created_at as created_since, and re-read a recent since/until window to catch voids and refunds. Insights: request date windows of up to 93 days with time_increment=day. Customers: follow next_cursor.
- Respect rate limits: read the RateLimit-Remaining headers and on HTTP 429 wait for Retry-After.
- Alternatively push events outward with outbound webhooks (see workspace-admin).
- Never put the API key in client-side code or commits.
