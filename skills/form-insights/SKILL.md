---
name: form-insights
description: Analyze an Inflowave account's forms — submission volume and trend, field drop-off, and (the important part) how form submissions convert downstream into leads, pipeline, and revenue. Flags leaks where people submit but get no follow-up, and offers to build the follow-up workflow. Use when the user says "how are my forms doing", "form conversion", "why aren't form leads converting", or asks about lead capture.
---

# Form Insights

Forms are the top of the funnel; the insight is in **what happens after** a
submission. Correlate form → lead → pipeline, find the leaks, and close them.

## Steps

1. **Scope.** `check_connection`; `list_sub_accounts` if needed. `list_forms` to
   inventory forms (name, active, submission count).

2. **Per form, read the funnel:**
   - `get_form_details` — fields and settings.
   - `get_form_submissions` — recent submissions, volume, trend.
   - Correlate downstream: for a sample of submitters, `search_leads` /
     `get_lead_details` / `get_lead_journey` to see whether the submission became
     a tracked lead, entered a pipeline, and moved.
   - `get_pipeline_analytics` — conversion + value for the pipeline those leads land in.

3. **Compute the leaks** (the correlations that matter):
   - **Capture leak** — submissions that never became a lead (integration/mapping issue).
   - **Follow-up leak** — leads created but **no workflow fired** on them
     (cross-check with `get_lead_active_workflows` / `get_lead_workflow_history`).
     This is the biggest, most common money leak.
   - **Stage leak** — leads that entered the pipeline but stalled at one stage.
   - **Field friction** — a field correlated with abandonment (if the data shows it).

4. **Quantify the opportunity.** Estimate value left on the table: (leads with no
   follow-up) × (this pipeline's average won value × its win rate). State it plainly.

## Output

- **Per form** — submissions (with trend), % that became leads, % that entered
  pipeline, % won.
- **Leaks** — ranked by estimated lost value, each with the cause.
- **The fix** — for the #1 leak (usually "no follow-up"), offer to build the
  follow-up automation now: a `form_submitted` trigger → tag → nurture sequence.
  If they say yes, hand off to the **build-workflow** skill (or build it inline
  with `create_workflow`, then `test_workflow`).

## Guardrails
- Read-only until the user approves building a workflow.
- Confirm the workspace before creating anything.
- Don't fabricate conversion numbers — if the downstream link can't be traced for
  a form, say the attribution is incomplete and show what you can.
