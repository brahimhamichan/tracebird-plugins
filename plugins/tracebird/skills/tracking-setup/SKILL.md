---
name: tracking-setup
description: "Makes Tracebird attribution and revenue trustworthy: installs the pixel, connects conversion sources (CRMs, webhooks, Calendly, Whop, Cortana), sends custom conversions, sets conversion definitions, Meta CAPI, the attribution model and front-end/back-end revenue settings. Use when the user says \"set up tracking\", \"numbers don't match\", \"connect my CRM\", \"pixel not working\", \"send sales to Meta\", \"fix attribution\"."
---
# Tracking and revenue setup

These tools change how conversions are attributed and what is sent to Meta. Show the current state first, explain the change, and confirm before each write.

## 1. Audit the current state
1. `get_workspace`, then `get_conversions_overview` (CRM and webhook connections, attribution model, last-7-day totals, recent events).
2. `get_pixel` (snippet, allowed domains, event counts, last install test), `get_revenue_settings`, `list_revenue_streams`, `list_offers`, `list_integrations`.
3. `list_conversions` (filter by type lead, appointment, show or sale, and by source) to see what actually arrives and whether events carry an ad, ad set or campaign.
4. Summarize gaps: no pixel, unattributed sales, missing revenue source, stale connections.

## 2. Pixel
1. `update_pixel_domains` with the website domains (domain only, for example example.com; it creates the pixel on first save).
2. `get_pixel` returns the snippet to paste in the site head. Give it to the user exactly.
3. `start_pixel_install_test`: open a returned test URL on the site with the snippet, then poll `get_pixel` (with pauses) until the install test shows seen.

## 3. Conversion sources
- GoHighLevel, HubSpot: browser sign-in. `get_conversion_source_connect_url` returns a connect_url for the user to open while signed in to Tracebird; then watch `get_conversions_overview`. Historical imports run in steps (`step_ghl_import`, `step_hubspot_import`, `choose_ghl_import`, `choose_hubspot_import`, `resume_ghl_import`, `resume_hubspot_import`); `disconnect_ghl`, `disconnect_hubspot` remove them.
- Custom webhook, Zapier, Make: `create_inbound_webhook` (name, provider, default_event_type) returns a receive URL for the sending tool; `rename_inbound_webhook_slug`, `delete_inbound_webhook`.
- Calendly: `connect_calendly` with a Personal Access Token (paid plan); `disconnect_calendly`.
- Whop: connect_url via `get_conversion_source_connect_url`, or `connect_whop_with_api_key`; `sync_whop`; `disconnect_whop`.
- Cortana: `create_inbound_webhook` with provider cortana, then `connect_cortana_api` for history, `import_cortana_history`, `remove_cortana_api`.
- Stripe: sign-in via `get_integration_connect_url`; `list_stripe_connections`.
- Never print API keys back; ask the user to paste keys only when needed.

## 4. Custom apps: record_conversion
`record_conversion` sends one conversion (type lead, appointment, show or sale; sales need value above 0; optional external_id for de-duplication, currency, email, phone, occurred_at not in the future). Attribute it with tracebird_visitor_id from the pixel, explicit ad_id / ad_set_id / campaign_id, UTM names or the contact's earlier lead. Test with one event, then confirm with `list_conversions`.

## 5. Conversion definitions and Meta CAPI
1. `list_conversion_sources_for_definitions`, `list_conversion_definitions`.
2. `create_conversion_definition` (name, event_types, optional sources and labels, meta_event_name); `update_conversion_definition`, `set_conversion_definition_enabled`, `duplicate_conversion_definition`; `delete_conversion_definition` only with confirmation.
3. Meta server-side events: `list_meta_capi_datasets`, `get_meta_capi_settings`, then `save_meta_capi_settings` (dataset, send_leads, send_purchases, optional test_event_code). Verify with `get_conversion_activity_feed` and `list_meta_capi_deliveries`; `retry_meta_capi_delivery` for failures.

## 6. Attribution model
`get_attribution_model` then `set_attribution_model` (first_click or last_click). This re-credits past conversions and changes ad-level numbers everywhere: state before and after and confirm. `get_attribution_report` compares Meta attribution windows.

## 7. Revenue settings (front-end / back-end)
1. `list_offers`: tag each offer front_end or back_end, or auto (price threshold decides) with `set_offer_stage`.
2. `list_revenue_streams`: choose what each stream counts as with `set_revenue_stream_role` (split, threshold, front_end, back_end, none, auto).
3. `update_revenue_settings`: back-end threshold, revenue attribution model (last_click, first_click, smart) and revenue source of truth (auto, stripe, crm, facebook).
4. Each change alters FE/BE/blended revenue and ROAS on dashboards. Compare `get_revenue` before and after.

## 8. Verify
Re-run `list_conversions` and `get_performance_summary`; confirm events arrive, carry attribution, and totals reconcile with the payment source. Report what is fixed and what remains.
