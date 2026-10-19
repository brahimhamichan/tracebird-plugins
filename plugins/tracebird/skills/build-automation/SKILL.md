---
name: build-automation
description: "Designs, previews, creates and manages Tracebird automation rules (auto-pause, scale budgets, alerts, renames) safely: schema, preview, create OFF, confirm, turn ON, folders, edits, run history and conflicts. Use when the user says \"automate\", \"create a rule\", \"auto-pause ads\", \"scale winners automatically\", \"alert me when CPL is high\"."
---
# Build an automation rule

Rules run on a schedule against synced ads and can change live ads. Always create OFF, preview, and confirm before turning ON.

## 1. Understand the goal
1. `get_workspace` for currency, timezone, scopes (needs write).
2. Pin down: which assets (campaign, ad set or ad; which accounts), the condition with thresholds and time window, the action (pause, change budget, notify), the schedule, and the guardrails.
3. Check what exists: `list_automation_rules`, `list_automation_folders`. Do not duplicate or oppose an existing rule. `list_ad_accounts` gives account ids for the filter.

## 2. Draft the config
1. Call `get_automation_rule_schema` before writing anything. It lists the config shape, metric keys, operators, actions, schedules and complete examples. Use only metric keys from it.
2. The config is { rule, filter, schedule }. Start from the closest example. Keep write_back_mode as dry_run while testing (it records what would change without touching the ad account); approved applies actions to the real ad account.
3. For budget actions set guardrails: max_budget_change_percent, max_daily_budget, max_budget_actions_per_day.
4. Prefer a notification action first when the user is unsure.

## 3. Preview
1. `preview_automation_rule` with the config. Read issues and warnings first and fix them.
2. Show the user which assets match (up to 100) and what the actions would change. Sanity-check count and spend of matched assets; refine the formula if it matches too much or nothing.

## 4. Create OFF
1. `create_automation_rule` with name, config, optional folder_id, status OFF (the default).
2. Show a summary: scope, condition in plain words, action, schedule and timezone, write_back_mode, guardrails, preview result.
3. Ask for explicit confirmation to turn it on. State whether it will act live (approved) or only record (dry run).

## 5. Turn ON
`set_automation_rule_status` with ON after confirmation. If it returns 409 the rule opposes another enabled rule on overlapping assets (the message names the conflicting rule): explain which one, then narrow the filter, change the action, or turn the other rule OFF with the user's approval. Do not work around it.

## 6. Organize and edit
- Folders: `create_automation_folder`, `rename_automation_folder`, `move_automation_rules` (folder_id "root" to remove from a folder), `delete_automation_folder` (confirm).
- Edit: `get_automation_rule`, change the config, `preview_automation_rule` again, then `update_automation_rule`. A rule that is ON keeps running with the new config and saving can 409 on a conflict, so confirm first.
- Copy: `duplicate_automation_rule`. Remove: prefer OFF over `delete_automation_rule`.

## 7. Monitor
- `run_automation_rule` evaluates now (confirm first; live if approved).
- `list_automation_runs` shows matched, changed, dry-run flag and per-asset details; `get_automation_history_policy` gives the retention.
- `list_automation_priority` shows queued or uncertain actions; these block manual edits until settled (`reconcile_automation_priority`, admin, only after the user verified in the ad account).
- Review the first runs with the user, then adjust thresholds.
