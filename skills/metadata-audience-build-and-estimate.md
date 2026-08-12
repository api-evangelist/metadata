---
name: metadata-audience-build-and-estimate
description: Choose the right Metadata audience constructor for a stated targeting intent, size it before committing spend, and keep the work reversible.
api: Metadata MCP Server
generated: '2026-08-12'
method: generated
source: https://metadata.io/developers/tools/
operations:
  - search_target_group_criteria
  - estimate_target_group
  - create_target_group
  - create_firmographic_audience
  - create_technographic_audience
  - create_bombora_audience
  - create_g2_intent_dynamic_audience
  - upload_account_list_csv_audience
  - get_deep_audience_details
  - archive_audience
---

# Build and size a Metadata audience

Metadata ships 31 audience and targeting tools. Picking the wrong constructor is the most common way an agent wastes credits here, because audiences are created at 5 credits each and the constructors are not interchangeable.

## Pick the constructor from the intent

| The user said | Tool |
|---|---|
| industry, headcount, revenue, country, job title, seniority | `create_firmographic_audience` |
| "companies running Salesforce / HubSpot / <vendor>" | `create_technographic_audience` |
| "people in market", Bombora intent topics | `create_bombora_audience` (see `get_intent_topics` first) |
| "researching us on G2" | `create_g2_intent_dynamic_audience` (live) or `create_g2_intent_static_audience` (snapshot) |
| "target this LinkedIn/Meta segment natively" | `create_linkedin_native_criteria_audience` / `create_facebook_native_criteria_audience` |
| "here is our account list / contact list" | `upload_account_list_csv_audience` / `upload_contact_list_csv_audience` (300–300,000 contacts, inline array or public CSV URL) |
| "people who already visited" | `create_retargeting_audience` |
| "use the segment we saved" | `create_audience_from_segment` (list them with `list_segments`, inspect with `get_segment_criteria`) |
| Reddit | `create_reddit_target_group` (search options with `search_reddit_criteria`) |

## Always size before you build

1. `search_target_group_criteria` — discover which criteria values actually exist. Do not guess enum values.
2. `estimate_target_group` — get the reach estimate.
3. Report the estimate to the user. If it is too small to spend against, adjust criteria and estimate again. Estimating is cheap; creating is not.
4. Only then call the constructor.

## Inspect and reverse

- `get_audience_details` for a summary, `get_deep_audience_details` for the full definition, `get_matched_audiences` to see what actually matched.
- `get_custom_audience_contacts_csv_url` to export a contact-list audience.
- Audience creation is documented as reversible: `archive_audience` and `unarchive_audience` rather than a hard delete. `write:audiences` is the only scope needed and it is not a destructive scope — this is the safest part of the Metadata surface for an agent to work in unsupervised.

## Guardrails

- `write:audiences` grants this whole flow. It does NOT grant campaign launch — that is `launch:campaigns`.
- Reads cost 1 credit, creates 5. `estimate_target_group` is the cheap way to be right.
