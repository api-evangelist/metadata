---
name: metadata-launch-multichannel-campaign
description: Build a B2B paid campaign in Metadata end to end — brand kit, audience, creative, offer, campaign draft — and hand the launch decision to a human before any spend moves.
api: Metadata MCP Server
generated: '2026-08-12'
method: generated
source: https://metadata.io/developers/tools/ + https://metadata.io/developers/guides/first-campaign.html
operations:
  - generate_brand_kit
  - create_firmographic_audience
  - estimate_target_group
  - generate_brand_creative
  - create_update_image_ad
  - create_update_offer
  - create_budget_group
  - create_campaign
  - add_and_edit_campaign_elements
  - launch_campaign
---

# Launch a multi-channel Metadata campaign

Every tool named here is a real tool in Metadata's published 141-tool catalog. Nothing is invented.

## Before you start

- Endpoint `https://mcp-server.metadata.io/mcp`, transport streamable-HTTP, MCP protocol `2025-06-18`.
- Auth is a bearer API key in `METADATA_API_KEY` (prefix `md1_live_`). It is scoped to ONE account.
- If the user has several accounts, call `list_user_accounts` first and pass `X-Account-ID` on every later call. A wrong id returns **403 `account_mismatch`**.
- Check your key's scopes. Drafting needs `write:audiences`, `write:creatives`, `write:campaigns`. Launching needs `launch:campaigns` — a separate, destructive scope. Missing it returns **403 `scope_denied`**.
- Every call costs credits: reads 1, creates/updates 5, launches and creative generation 10–25. A zero balance returns **402 `insufficient_credits`**.

## Steps

1. **Ground yourself.** `get_account_details` for the account name, plan and connected channels; `get_integrations_status` to confirm the channels the campaign will use are actually connected. If a channel is missing, stop — `connect_channel` is destructive and needs `write:integrations` plus a human.
2. **Extract the brand.** `generate_brand_kit` against the customer's domain, or `get_brand_kit` if one already exists. This is what keeps generated creative on-brand.
3. **Build the audience.** Pick the right constructor for the targeting the user described, not the first one you see:
   - firmographic (industry, employee count, revenue, geography, titles) → `create_firmographic_audience`
   - technology stack → `create_technographic_audience`
   - Bombora intent → `create_bombora_audience`; G2 intent → `create_g2_intent_dynamic_audience` or `create_g2_intent_static_audience`
   - a supplied contact or account list → `upload_contact_list_csv_audience` / `upload_account_list_csv_audience` (300–300,000 contacts)
   - re-engagement → `create_retargeting_audience`
4. **Size it before you commit.** `search_target_group_criteria` to discover valid criteria, then `estimate_target_group` to check the audience is big enough to spend against. Report the estimate to the user before building anything else.
5. **Generate creative.** `generate_brand_creative` (or `generate_flexible_brand_creative`) using the brand kit, then wrap it in an ad with `create_update_image_ad` / `create_update_video_ad` / `create_update_carousel_ad` as the format requires.
6. **Attach a destination.** `create_update_offer`. Lead Gen offers are locked to the channel they were created for — build one per channel. `find_offer_url` and `find_privacy_url` help resolve the URLs.
7. **Set the money guardrail first.** `create_budget_group` with the monthly cap and goal (CPL for lead gen, CTR/CPC for brand). Do this BEFORE creating the campaign so the campaign is born inside a cap.
8. **Create the campaign in Draft.** `create_campaign` for the default Precision (1x1x1) structure, where each audience × ad × offer combination runs as its own experiment; `create_native_structure_campaign` for the Channel-First (NxNxN) layout when the user wants the ad platforms' native optimization. Campaigns are Draft by default and spend nothing.
9. **Assemble.** `add_and_edit_campaign_elements` to attach audiences, ads, offers and keywords per channel (use `add_and_edit_native_campaign_elements` for Native campaigns — they are not interchangeable).
10. **STOP. Hand off.** `launch_campaign` transitions Draft → LIVE and real budget starts spending on real ad platforms. Do not call it autonomously. Present the campaign structure, the audience estimate, the creative, the offer and the budget cap, and get an explicit human go-ahead from the person who owns paid media.

## After launch

`experiment_performance_stats` and `deep_funnel_stats` to read results; `update_experiments_daily_budgets` to shift spend toward what is working; `manage_campaign` to pause or resume.

## Conventions that apply throughout

- Metadata documents destructive actions such as `launch_campaign` and `connect_crm` as idempotent, but publishes no client-supplied idempotency key — you cannot make an arbitrary retry safe on your own terms. On an ambiguous failure, read state back (`get_campaign_by_wizard_id`, `search_campaigns_by_names`) before retrying a create.
- No rate limits are published. Back off on your own.
- Errors are a vendor code table, not RFC 9457: `missing_key`/`invalid_key` (401), `scope_denied`/`account_mismatch` (403), `insufficient_credits` (402). See `errors/metadata-problem-types.yml`.
