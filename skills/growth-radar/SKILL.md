---
name: growth-radar
description: A cross-domain correlation engine for an Inflowave account — pulls engagement, forms, workflows, pipeline, and revenue together to find what actually drives outcomes (and what's silently leaking money), then ranks concrete opportunities with the exact next action for each. Use when the user says "where's my biggest opportunity", "what should I focus on", "find growth", "what's leaking", or wants prioritized suggestions rather than a report.
---

# Growth Radar

The suggestion engine. A weekly recap tells you *what happened*; Growth Radar
tells you *what to do next and why*, by correlating across domains that are
normally looked at in isolation.

## Method

1. **Scope.** `check_connection`; confirm workspace via `list_sub_accounts`.

2. **Gather signals across all domains** (reads, parallel):
   - Audience/reach: `get_engagement_analytics`, `get_follower_analytics`, `get_account_insights`.
   - Capture: `list_forms` + `get_form_submissions`; `list_leads` / `search_leads` by source.
   - Automation: `list_workflows`, `get_workflow_performance`, `workflow_health_audit`, `get_workflow_failure_stats`.
   - Conversion: `get_pipeline_analytics`, `get_pipeline_board`, `list_pipeline_opportunities`.
   - Money: `get_finances_summary`; attribution via `get_lead_journey` on a sample.
   - Reach amplifiers: `get_link_analytics` (which links/CTAs actually get clicked).

3. **Correlate — this is the whole point.** Look for relationships, not totals:
   - **Source → revenue.** Which lead source (form, DM, campaign, link) produces
     the highest *won value per lead*, not just the most leads? Fund the winner.
   - **Workflow → revenue.** Which active workflow's contacts convert best? Which
     "busy" workflow drives lots of activity but no wins?
   - **Leak points.** Biggest drop between stages: submitted→lead, lead→no-follow-up,
     stage→stage. Quantify each in lost $ (volume × downstream win value).
   - **Underused wins.** A high-converting workflow that only runs on a fraction of
     eligible leads; a high-click link buried where few see it.
   - **Silent failures.** A source that dried up, an automation that stalled, an
     integration that disconnected.

4. **Rank opportunities** by `estimated_value × confidence ÷ effort`. Estimate the
   $ upside for each from the pipeline's own average won value and win rate — and
   be honest about confidence.

## Output

A ranked **Opportunity Radar** — 3 to 5 items, biggest first. Each item:

- **The opportunity** (one line, in money terms where possible):
  *"~$4k/mo left on the table: 120 form leads/mo get no follow-up."*
- **Why** (the correlation you found).
- **Do this** (the exact next action — often "build workflow X" or "shift budget
  to source Y"), with an offer to do it now (hand off to **build-workflow** or
  **workflow-doctor**).

## Guardrails
- Read-only analysis; any action is offered, never taken automatically.
- Every suggestion must trace to a correlation you actually observed — no generic
  advice ("post more"). If the data is too thin to correlate, say so and name
  what to instrument first.
- Money estimates are clearly labeled estimates with their assumption stated.
- Plain language, ranked, skimmable. Lead with the single biggest lever.
