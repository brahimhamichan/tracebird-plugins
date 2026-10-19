---
name: workspace-admin
description: "Manages Tracebird workspace settings (name, timezone, currency), team members and invitations, notifications, logs and audit trail, outbound webhooks and billing status. Use when the user says \"change workspace timezone/currency\", \"invite a teammate\", \"remove a member\", \"show the audit log\", \"what happened in the logs\", \"add a webhook\", \"what plan am I on\"."
---
# Workspace administration

Most writes here need an admin. Explain consequences, confirm, then act. Never expose secrets.

## 1. Orient
`get_workspace` (name, currency, timezone, plan, acting user, scopes). `get_workspace_settings` for profile, website, main currency, timezone and revenue source preference. If a call is refused as admin only, tell the user they need an admin to do it or to be promoted.

## 2. Workspace settings
`update_workspace` changes name, website_url, currency, timezone, revenue_source_preference. Send only the fields to change.
- Currency and timezone change how every report and dashboard total and day boundary is shown. Warn that day totals shift, comparisons with exports may no longer match, and scheduled reports use their own timezones. Show before and after, then confirm.
- Personal preferences (not workspace-wide): `get_user_settings` and `update_user_settings` (default timeframe, landing page, notification toggles, automation defaults).

## 3. Team
1. `list_team_members` (roles admin or manager, owner, seat usage against the plan) and `list_team_invitations`.
2. `invite_team_member` (email, role admin or manager): emails an invitation valid for seven days; needs a plan with team access. Confirm email and role.
3. `resend_team_invitation`, `revoke_team_invitation`.
4. `update_team_member_role` (cannot demote yourself or the owner) and `remove_team_member` (immediate; confirm name and consequences).
5. Check seats first so the invite does not fail on plan limits.

## 4. Notifications
`get_notification_summary` for unread count and latest, `list_notifications` (filter unread or by category), then `mark_notification_read`, `mark_all_notifications_read` or `dismiss_notification`. Unread integration or automation alerts usually explain stale data.

## 5. Logs and audit
- `list_logs`: integration syncs, automation runs, team actions, API and MCP changes, operational events; filter by category, status, since, q; paginate with next_cursor.
- `list_audit_log`: security and workspace events (keys, webhooks, member actions, changes made through the API or MCP).
- `list_sync_history` for sync runs per provider. Use these to answer "who changed X" and "why did Y fail"; quote timestamps and actors.
- Credentials: `list_api_keys` and `list_connected_apps` show metadata only; keys and app connections are created or revoked in the Tracebird app (Settings, API and MCP).

## 6. Outbound webhooks
1. `list_webhooks` (revoked ones included; secrets never shown again).
2. `create_webhook` with name and an HTTPS endpoint_url (admins; max 25 active). Confirm the endpoint. The signing secret is returned exactly once: show it to the user, tell them to store it, and never repeat or save it.
3. `revoke_webhook` stops deliveries; confirm first.

## 7. Billing status
`get_billing`: plan, status (demo, trialing, active, past_due, canceled), trial end, seat and ad account limits, trailing 30-day ad spend versus allowance. Read-only: checkout, plan changes and invoices happen in the Tracebird app under Billing; give the user that pointer. Flag past_due, canceled or near-limit usage.
