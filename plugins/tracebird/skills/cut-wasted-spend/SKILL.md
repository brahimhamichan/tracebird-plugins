---
name: cut-wasted-spend
description: "Finds ads and ad sets that spend without results (zero purchases, high CPA, low ROAS) and, after the user confirms, pauses them or shifts budget. Use when the user says \"cut wasted spend\", \"what should I pause\", \"stop losing money\", \"find underperforming ads\", \"reduce CPA\"."
---
# Cut wasted spend

This skill changes live ad accounts. Analyze first; change only after explicit confirmation.

## 1. Context
1. Call `get_workspace` (currency, timezone, write scope). If the connection is read-only, produce the analysis only and explain that write access needs a reconnect.
2. Ask for or choose targets: break-even or target CPA/ROAS, minimum spend before judging (default: at least 2 to 3 times the target CPA), and the window (default last 7 days, plus the last 3 days for trend).

## 2. Find waste
1. `get_insights` at level ad and level adset for the window, time_increment total. Page through next_cursor. Use `get_revenue` or `get_funnel_asset_metrics` so back-end revenue counts and a front-end loss-maker that earns back-end revenue is not paused by mistake.
2. Flag, per entity (include id, name, spend, results, CPA, ROAS):
   - Spend above the threshold with zero results.
   - CPA above target by a clear margin, or ROAS below break-even, for the whole window and the last 3 days.
   - Do not flag: entities still in the learning phase, with tiny spend, or whose results come in late (check the attribution lag).
3. Verify status with `list_ads` or `list_ad_sets` (filter by campaign_id) so you only propose changes to ACTIVE entities. Use `get_ad`, `get_ad_set`, `get_campaign` for details.
4. Estimate impact: spend recovered per day if paused, or budget moved to the best performers.

## 3. Propose
Present a table: entity, level, id, current status or budget, proposed change (pause, or daily budget before to after), evidence numbers, expected impact. Prefer pausing over deleting. For budget shifts name the winner receiving the budget. Ask: "Apply these N changes?".

## 4. Execute (only after confirmation)
- `update_ad` with status PAUSED (or ACTIVE to revert).
- `update_ad_set` with status or daily_budget.
- `update_campaign` with status or daily_budget.
- Budgets are in major units of the ad account currency. Send one field kind per request (status or name or budget as the tool allows) and one entity per call.
- Changes are live immediately. Do not use `delete_ads`.

## 5. Automation priority and 409
- If an active automation rule has priority over the entity, the change is refused with HTTP 409 and an explanation. Do not retry blindly: tell the user which rule is involved (see `list_automation_rules`, `get_automation_rule`) and offer options: edit or turn off that rule with `update_automation_rule` / `set_automation_rule_status` (needs confirmation), or leave the entity to the rule.
- `list_automation_priority` shows assets with a queued or uncertain automation action; manual edits stay blocked while pending. For an uncertain item, the user must first check the ad account; only then `reconcile_automation_priority` (admin) after a fresh sync.
- A budget that lives at campaign level (budget optimization) cannot be set on the ad set; the tool says so. Change it on the campaign instead.

## 6. Report
After applying, list each change with result (applied, refused and why). Re-read with `get_ad_set` or `list_ads` to confirm status. Suggest a recheck date (for example 3 days) and build-automation for a recurring guardrail. Mention how to undo: set status back to ACTIVE or restore the old budget (give the old value).
