---
name: weekly-recap
description: Run a narrated weekly business review of an Inflowave account — growth, engagement, pipeline, revenue, and the top-performing (and stalling) workflows — ending in three prioritized moves. Use when the user says "weekly recap", "how did we do this week", "give me my numbers", or asks for a business summary.
---

# Weekly Recap

Produce a concise, plain-English weekly review for a business owner. Never dump
raw tool output — synthesize it into a story with a cause for every number and a
short ranked action list at the end.

## Steps

1. **Establish scope.** Call `check_connection` to confirm the account, plan, and
   what's available. If the user manages multiple workspaces, call
   `list_sub_accounts` and ask which one (or "all") — default to the whole agency
   only if they say so.

2. **Pull the headline numbers** (all reads, safe to run in parallel):
   - `weekly_recap` — the platform's own rollup if available; use it as the spine.
   - `get_dashboard_summary` — top-line KPIs.
   - `get_engagement_analytics` and `get_follower_analytics` — audience + reach movement.
   - `get_pipeline_analytics` — opportunities created / won / lost / value.
   - `get_finances_summary` — revenue and spend.
   - `get_team_performance` — if they have a team, who moved the needle.

3. **Find the movers, not just the totals.** For every metric that changed
   meaningfully, name the *why*: a spike in leads → check the source (form,
   campaign, DM); a jump in won revenue → which pipeline / workflow; a drop in
   replies → which account or workflow stalled. Pull one supporting read to
   confirm the cause before asserting it.

4. **Surface the top and bottom workflows.** `get_workflow_performance` (or
   `list_workflows` + `get_workflow_details`) to name the 1–2 workflows driving
   the most outcomes and any that quietly stopped firing.

## Output

A short brief, in this shape:

- **This week in one line** — the single most important thing that happened.
- **Growth** — audience/reach, with the driver.
- **Pipeline & revenue** — created vs won, value, and the source of the wins.
- **Automations** — the best performer and anything that stalled.
- **Do this next** — exactly 3 ranked actions, each one sentence, each tied to a
  number above (e.g. "Re-activate the 'New Lead DM' workflow — it drove 40% of
  replies last week and has been off for 3 days").

## Guardrails
- Read-only. Never create or change anything in this skill — if an action is
  warranted, name it in "Do this next" and let the user ask.
- Plain language. No tool names, no p95s, no internal IDs in the output.
- If a number looks off (zero, or a huge swing), say so honestly rather than
  inventing a cause.
