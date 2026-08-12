---
name: metadata-performance-and-pipeline-reporting
description: Read Metadata campaign, funnel and account-level performance with read-only tools and report on pipeline rather than lead volume.
api: Metadata MCP Server
generated: '2026-08-12'
method: generated
source: https://metadata.io/developers/tools/
operations:
  - get_account_details
  - account_level_stats
  - account_funnel_reports
  - deep_funnel_stats
  - experiment_performance_stats
  - budget_group_performance
  - get_converted_leads
  - get_account_opportunities_insights
  - get_account_timeline_insights
  - performance_metrics
---

# Report on Metadata performance

This is the safe entry point to the Metadata MCP surface. Every tool here is read-only and covered by the single `read:all` scope, so an agent can do all of it without any ability to create, change or spend.

## Steps

1. `get_account_details` — account name, plan, connected channels, credits remaining. Do this first so every number you report is attributed to the right account.
2. For multi-account users, `list_user_accounts` and loop, passing `account_id` / `X-Account-ID` per account.
3. **Top line:** `account_level_stats` over the date range the user asked for. `performance_metrics` for the metric breakdown.
4. **Funnel:** `account_funnel_reports`, then `deep_funnel_stats` when the user wants stage-by-stage movement rather than a single conversion rate.
5. **Money:** `budget_group_performance` for spend against cap per budget group; `experiment_performance_stats` to find which audience × ad × offer experiments are earning their spend.
6. **Revenue, not leads:** `get_converted_leads` and `get_converted_leads_summary` for conversions; `get_account_opportunities_insights` for pipeline; `get_account_timeline_insights` for the account journey. Metadata's whole positioning is optimizing for pipeline over lead volume — report in that order.
7. **Search:** `list_search_terms` and `experiments_keywords_stats` for paid search; `demographic_country_stats` and `website_engagement_stats` for audience and site behaviour.
8. **Ad-hoc:** `search_insights_criteria_fields` to discover queryable fields, then `query_metadata_analytics_account`; `query_metadata_analytics_benchmarks` to compare against benchmarks.

## Reporting discipline

- Every read costs 1 credit and analytics reads average 3. Batch what you need in one pass rather than polling.
- State the date range and the account on every figure you report.
- If you want to act on what you find, that is a different scope and a different skill — budget changes go through `update_experiments_daily_budgets`, and anything that moves spend should be proposed to a human, not executed.
