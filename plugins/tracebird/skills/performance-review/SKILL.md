---
name: performance-review
description: "Runs a daily, weekly or monthly Tracebird performance review: spend, results, ROAS, front-end/back-end/blended revenue, winners, losers and anomalies versus the previous period. Use when the user says \"how are my ads doing\", \"weekly report\", \"performance review\", \"what's working\", \"why did ROAS drop\"."
---
# Performance review

Read-only analysis. Make no changes; end with recommended actions the user can approve.

## 1. Set scope
1. Call `get_workspace` for currency, timezone and scopes. Dates are YYYY-MM-DD in the workspace reporting timezone.
2. Pick the period: daily = yesterday, weekly = last 7 full days, monthly = last 30 days (ranges are limited to 93 days). Compute the previous period of equal length for comparison.
3. Call `list_ad_accounts` if the user has several accounts; ask which one(s) if it matters. Never add up amounts in different currencies.

## 2. Pull the numbers
1. `get_performance_summary` for the period and the previous period: spend, impressions, clicks, platform purchases and ROAS, attributed leads and revenue, CRM funnel counts, Stripe revenue.
2. `get_revenue` for the same two periods: front-end, back-end and blended revenue. Default basis is acquisition (revenue credited to the day the customer was acquired); recent ranges are lower bounds while maturing is true. Use basis sale to see cash by sale day.
3. `get_insights` with level campaign, then ad set and ad for the top spenders, time_increment total (use day for trends). Page with next_cursor.
4. Optional depth:
   - `get_insight_breakdowns` by placement, device, age, gender or country for a campaign or ad set that moved.
   - `get_creative_detail` and `get_creative_breakdowns` for one ad's daily series and its placement/age/device split.
   - `get_funnel_asset_metrics` for front-end, back-end and blended revenue (and LTV) per campaign, ad set or ad.
   - `get_dashboard_trend` for daily spend versus revenue.

## 3. Analyze
- Compute change versus the previous period for spend, revenue, ROAS, cost per result, CTR, CPC, CPM.
- Winners: high ROAS or low cost per result with enough spend to trust (say the spend threshold you used). Losers: spend with no or poor results.
- Anomalies: a day with a spend or revenue jump, CPM or CTR shifts, frequency creep, a campaign that stopped delivering.
- Platform-reported purchases and Tracebird attributed revenue differ by design; say which source each number comes from.
- Compare against the revenue settings: if blended and platform ROAS diverge a lot, check tracking before blaming ads.

## 4. Data-freshness caveats
- Check `list_integrations` or `list_sync_history` for last sync time and errors; state if data is stale.
- Today's and yesterday's numbers are incomplete; attribution lags (back-end sales arrive later).
- Ad-platform days follow each ad account's own timezone and can differ slightly from the workspace timezone.
- Small samples (under about 30 results) are noisy: say so.

## 5. Report format
1. Headline: spend, revenue (FE/BE/blended), blended ROAS, change versus previous period.
2. Table of top campaigns (spend, results, cost per result, ROAS, change).
3. Winners and losers (named, with ids and numbers).
4. Anomalies and caveats.
5. Recommended actions, ranked by expected impact. Point to cut-wasted-spend for pauses and budget shifts and build-automation to make a recurring check; do not change anything without confirmation.
