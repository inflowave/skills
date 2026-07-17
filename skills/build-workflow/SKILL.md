---
name: build-workflow
description: Build a new Inflowave automation workflow from a plain-English goal — pick the right trigger, actions, and wiring, create it as a draft that's valid by construction, then run a safe simulated test before the user activates it. Use when the user says "build a workflow", "create an automation", "set up a nurture sequence", "when X happens do Y", or describes an automation they want.
---

# Build Workflow

Turn "when a new lead comes in, tag them and send a welcome DM after an hour" into
a working, tested draft — without the user touching the builder.

## Steps

1. **Scope + workspace.** `check_connection`; `list_sub_accounts` and confirm
   WHICH workspace this workflow belongs to (never assume). If it should target
   specific clients, `list_clients` and confirm.

2. **Nail the trigger.** Map the user's "when" to a real trigger with
   `list_trigger_types`, then `describe_trigger` for its exact config shape.
   Common ones: `contact_created`, `form_submitted`, `tag_added`,
   `appointment_booked`, `instagram_message`, `stripe_payment_received`,
   `custom_object_record_created`. Confirm any config (keywords, form id, object type).

3. **Design the steps.** Map each "do this" to an action with `list_action_types`;
   for message/email/SMS/DM copy, use only **namespaced** template variables
   (`{lead.first_name}`, `{lead.email}`, `{agency.name}`) — call
   `list_template_variables` and never use a bare `{first_name}` (the engine
   rejects it). Insert delays/conditions where the flow needs them.

4. **Assemble the envelope and create.** Call `create_workflow` with:
   - `sub_account_id` (the confirmed workspace)
   - `name`, `trigger`, `nodes` (each `step_type` + typed `config`), `edges`.
   - The trigger node is auto-injected as `trigger_1` — wire your first node's
     incoming edge from `source_node_id: "trigger_1"`; do not include the trigger
     in `nodes`.
   - The first call returns a **confirmation envelope** echoing the resolved
     workspace/clients — relay it, confirm, then re-call with `confirm: true`.
   - It's created as a **draft** (`is_active: false`). Good — never auto-activate.

5. **Test before activation.** Run `test_workflow` — it simulates the run
   (messages are NOT sent to real people, waits are skipped) and returns a
   per-node trace. Walk the user through what would happen. Only activate if they
   explicitly ask.

## Output

- A plain-English description of the workflow you built (trigger → steps).
- The draft id.
- The test-run trace, summarized ("a new lead would be tagged, then DM'd after 1h
  with: …").
- "Ready to turn it on? Say the word and I'll activate it."

## Guardrails
- Valid by construction: use `describe_trigger` / `list_action_types` /
  `list_template_variables` so configs and variables are correct on the first try.
- Never activate without an explicit request.
- Confirm the workspace and any client scope before creating.
- If the goal needs a disconnected integration (IG, a domain, a payment
  provider), flag it — the workflow will build but won't fire until connected.
