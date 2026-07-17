---
name: workflow-doctor
description: Audit an Inflowave account's automation workflows — find broken, failing, stalled, or underperforming ones, correlate each problem to the exact node/step that's causing it, propose fixes, and (with the user's OK) apply and test them. Use when the user says "audit my workflows", "what's broken", "why isn't my automation working", "fix my workflows", or reports a workflow not firing.
---

# Workflow Doctor

Diagnose and repair workflows. The value is in **correlation** — don't just say
"this workflow has a low success rate," say *which node* fails and *why*, then
offer the specific fix.

## Steps

1. **Scope.** `check_connection`, then `list_sub_accounts` if multi-workspace and
   ask which one. `list_workflows` to get the inventory (name, trigger, active,
   execution count).

2. **Run the audit.** `workflow_health_audit` for the account-level view, then for
   each concerning workflow:
   - `get_workflow_performance` — success rate, avg duration, throughput.
   - `get_workflow_failure_stats` — what's erroring and how often.
   - `get_workflow_node_analytics` / `get_workflow_step_analytics` — **the key
     correlation**: which specific node drops or fails contacts.
   - `get_workflow_details` — the actual config of the failing node.

3. **Classify each problem** into one of:
   - **Broken** — errors at a node (bad config, missing integration, invalid
     template variable). Correlate the error to the node.
   - **Stalled** — active but hasn't fired recently; usually the trigger no longer
     matches (e.g. a tag/form that was renamed) or the source integration
     disconnected (`check_connection` on that account).
   - **Leaky** — fires fine but contacts drop at a wait/condition node that's too
     strict, or a send step with a low open/click.
   - **Dormant** — inactive draft that should probably be live.

4. **Propose a fix per problem** — concrete and specific: "Node 3 'Send Email'
   fails with invalid_template_variable `{first_name}` — should be
   `{lead.first_name}`." Use `list_template_variables`, `describe_trigger`,
   `describe_condition` to get the correct shapes.

5. **Apply on approval.** Only after the user says yes: update via the appropriate
   write tool (or `create_workflow` for a rebuild), then **always** `test_workflow`
   (simulated — nothing sent to real people) and report the per-node trace. Never
   activate a workflow the user didn't ask to activate.

## Output

- **Health summary** — X workflows, Y healthy, Z need attention.
- **Findings** — a ranked list, worst first. Each: workflow name → the exact node
  → the cause → the one-line fix. Rank by impact (contacts affected × how central
  the workflow is).
- **Proposed fixes** — numbered, each with the precise change. Ask which to apply.

## Guardrails
- Read + diagnose freely; **never write without an explicit yes** naming the
  workflow(s) to change.
- Confirm the workspace before any change (the MCP enforces this via a
  confirmation envelope — relay it).
- After any change, test in simulation before suggesting activation.
- If a fix needs something outside Inflowave (reconnect Instagram, verify a
  domain), say so plainly instead of pretending the workflow alone can fix it.
