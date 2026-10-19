---
name: split-testing
description: "Creates and runs landing-page split tests in Tracebird (variants, start/pause, report, winner) and sets up custom split-link domains. Use when the user says \"split test\", \"A/B test my landing page\", \"which page converts better\", \"declare a winner\", \"set up go.mybrand.com for split links\"."
---
# Split testing

Split tests send traffic arriving on one public split link across 2 to 5 landing-page variants and measure leads, appointments, shows, sales and revenue per variant. Starting and declaring a winner affect real visitors.

## 1. Preconditions
1. `get_workspace` (scopes). Run `list_split_tests` to avoid duplicating a test.
2. Results need tracking on the landing pages: the Tracebird pixel (`get_pixel`) or CRM linking. If missing, do the tracking-setup skill first, otherwise the report will show no outcomes.
3. Ask: test name, primary metric (lead, appointment, show, sale or revenue), variant URLs (http/https, must not be split links), traffic weights, and the ad destination plan (the ad should point at the split link).

## 2. Optional: custom link domain
1. `list_split_domains` for existing domains and which is default.
2. New one: confirm the hostname (for example go.brand.com) with the user, then `connect_split_domain`. It returns the CNAME and TXT records the user must create at their DNS host. Show them exactly.
3. `detect_split_domain_dns` tells where DNS is hosted. If Cloudflare and the user supplies a scoped API token (DNS edit on the zone; used once, never stored), `apply_split_domain_cloudflare` can create the records; confirm first.
4. After the records exist, `refresh_split_domain_verification` (throttled to one check per 10 seconds; DNS can take minutes). Repeat until the domain is active, with sensible waits.
5. `update_split_domain` renames it or makes it the default; `remove_split_domain` only with confirmation if no test uses it.

## 3. Create (draft)
1. `create_split_test` with name, primary_metric, variants (name, url, weight; weights are relative integers), optional slug, domain_id, assignment_mode, pass_query_params (keeps UTMs and click ids), auto_stop. Check the tool's parameter descriptions for allowed values.
2. It creates a draft and returns the public split link. Nothing is served yet. Variants and assignment mode can only change while it is a draft (`update_split_test`).
3. Have the user open the link and each variant to check it loads and the pixel fires.

## 4. Start and pause
- `start_split_test` begins sending live traffic (also resumes a paused test). Confirm first.
- `pause_split_test` stops splitting. Use `duplicate_split_test` to rerun with changes.
- `get_split_test` returns the definition; `get_split_test_report` the results.

## 5. Read results with caution
1. `get_split_test_report`: per variant visitors, conversions, revenue per visitor, probability to beat control, relative lift with a 95% interval, verdict and a sample-ratio-mismatch check.
2. Rules of thumb:
   - Do not call a winner early; wait for the verdict and enough conversions per variant (dozens at least), and a full week or business cycle.
   - If the sample-ratio check fails, traffic was not split as configured: investigate (redirects, caching, bots, tracking) before trusting results.
   - A wide interval that includes zero means no proven difference.
   - Revenue and sale metrics need more data than leads. Check that attribution is mature.
3. Summarize: leader, lift with interval, confidence, sample sizes, caveats, recommendation.

## 6. Declare a winner
`declare_split_test_winner` with the winning variant_id completes the test and sends 100% of the link's traffic to it. It cannot be undone: confirm the variant name, evidence and consequence. Afterwards `delete_split_test` only if the user asks.
