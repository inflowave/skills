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

## Node + edge shape (get this right or `create_workflow` 422s)

The MCP validator is STRICTER than the loose examples in `get_workflow_schema`.
Build every node and edge in exactly this shape:

- **Node**: `{ "id", "name", "step_type", "config": { "action_type", ...fields } }`.
  - The action verb goes ONLY inside `config.action_type` — a top-level
    `action_type` is rejected ("extra_forbidden").
  - Do NOT send `position` — it is rejected too (the builder lays nodes out).
  - `step_type` is one of `action | condition | delay | wait_for_response |
    router | switch | split | merge | end` (see `get_workflow_schema` steps).
- **Edge**: `{ "id", "source_node_id", "target_node_id", "condition_label"? }`.
  - Do NOT send `condition_value` — rejected. Branch ONLY on `condition_label`.
  - For a boolean `condition` node, the two edges MUST read as true/false:
    use `condition_label` `"true"` and `"false"` (synonyms `yes/no`, `1/0`,
    `t/f` also work). A condition with only one branch FAILS CLOSED — the path
    just ends, it does not fall through.
  - For a `wait_for_response` node, label the edges `"responded"` and
    `"timeout"`.
- **Trigger**: pass it as the top-level `trigger` arg (NOT in `nodes`). It is
  auto-injected as node id `trigger_1`; wire your first node's incoming edge from
  `source_node_id: "trigger_1"`.

Validate before handing off: `validate_workflow_config(workflow_id, ...)` returns
`is_valid` + `can_activate` and is the source of truth. Note that `test_workflow`
reports `skipped` for a keyword-filtered comment/DM trigger — the harness's
synthetic event doesn't contain the keyword, so the trigger correctly doesn't
fire. That is NOT a broken workflow; confirm with `validate_workflow_config`.

## Comment-to-DM playbook (Instagram)

The single most-requested automation. Build it like this:

1. **Trigger** = `comment_posted` (frontend alias `post_or_reel_comment`).
   Config: `keywords` (list), `keyword_match: "any"`, optionally `post_id` /
   `specific_posts` to limit to one post, `include_replies` to also fire on
   reply-comments. Runtime context you can use in copy: `{trigger.comment_text}`,
   `{trigger.comment_id}`, `{trigger.commenter_id}`, `{trigger.post_id}`.
2. **First DM = a private reply, automatically.** Use a normal `send_dm` node.
   For a commenter who never messaged first, Instagram forbids a cold DM — the
   engine detects the cold-comment open and sends that first message as an IG
   **private reply to the comment** (allowed once per comment, 7-day window). You
   do NOT need a special action; just `send_dm` off a comment trigger.
3. **Buttons that you can branch on must be `postback`**:
   `"buttons": [{ "type": "postback", "title": "...", "payload": "..." }]`. The
   `clicked_button` condition matches on that `payload`. A `type: "url"` button is
   UNTRACKABLE (Meta sends no webhook on tap) — to gate on a link click, use a
   tracked link + a `wait_for_link_click`, never `clicked_button`.
4. **Point-in-time conditions need a wait in front of them.** `clicked_button`,
   `message_seen`, `follows_account` and `lead_has_responded` evaluate instantly,
   so a condition placed right after a send always takes the NO branch (the human
   hasn't acted in that split second). Put a `wait_for_response` (resolves the
   instant they tap/reply) or a `delay` BEFORE the condition.
5. **Public reply vs DM**: `reply_comment` posts a public reply (requires
   `message_template`); `send_dm` opens the private thread. A great combo does
   both — public acknowledgement, then the resource in the DM.
6. **AI replies**: `ai_response` (config `ai_instructions`, optional
   `message_content: "{trigger.comment_text}"`) produces `ai_message`; feed it
   into a `reply_comment` or `send_dm` via `{workflow.<ai_node_id>.ai_message}`.

### Advanced gates — what's real vs best-effort
- **Follower-count gate** (e.g. VIP > 1000): reliable. Add a `fetch_profile_info`
  action first to populate the profile, then a `follower_count_greater_than`
  condition (`operand: 1000`). VIP branch → `send_voice_note`, standard branch →
  `send_dm`.
- **Voice note**: `send_voice_note` accepts `text` alone (the account's default
  voice is used); for the customer's own cloned voice add `voice_clone_id` from
  `list_voice_clones`. Tell them to record a clone for the personal touch.
- **Follows-us check**: `follows_account` reads a profile flag that is only
  populated AFTER `fetch_profile_info` (or an inbound profile webhook) has run —
  put it behind a wait. Pairing it with a "Do you follow us?" `postback` buttons
  ask gives a good UX and a reliable signal.
- **Country / region filter**: `from_country` + operator `in_list` / `not_in_list`
  + a list operand EXISTS, but Instagram does not expose a commenter's country,
  so `{lead.country}` is usually EMPTY for a fresh commenter. Design the branch so
  an UNKNOWN country PROCEEDS (don't hard-block on empty), and tell the customer
  it only filters leads whose country was enriched by other means. Don't oversell
  it as reliable geo-blocking.
- **Collect email**: there is no dedicated capture action. Pattern: `send_dm`
  asking for the email → `wait_for_response` → `has_email` condition (or persist
  with `update_lead`, `data: { "email": "{workflow.<wait_node>.response.response_content}" }`),
  then `add_tag` to mark them captured.
- **Tag responders** at each meaningful step with `add_tag`
  (`tag_names: ["..."]`) so the customer can segment and retarget.

Reference builds live in the Inflowave workspace: "Comment to DM — Keyword
Auto-Reply", "Comment Response — AI Public Reply", "Comment to DM + Public Reply
(Combo)", and "VIP Comment-to-DM Engine (Advanced)".
