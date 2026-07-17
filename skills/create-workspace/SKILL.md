---
name: create-workspace
description: Create a new workspace (sub-account) inside an Inflowave agency — with the right operating mode (done-for-you / done-with-you / do-it-yourself), optional multi-client swarm seats, and data region — confirming the name and settings before creating. Use when the user says "create a workspace", "add a client workspace", "spin up a sub-account", or "set up a new workspace for X".
---

# Create Workspace

Inflowave agencies are multi-tenant: an agency contains workspaces (sub-accounts),
and each workspace owns its own accounts, leads, workflows, and campaigns. This
skill provisions one cleanly.

## Steps

1. **Scope + dedupe.** `check_connection` (must be an agency owner on a plan that
   allows managing workspaces — restricted/free tiers are blocked). `list_sub_accounts`
   to make sure a workspace with this name doesn't already exist.

2. **Gather the essentials** (ask only for what you don't know):
   - **name** (required) — the workspace / client name.
   - **operating_mode** — `dfy` (done-for-you: the agency runs it),
     `dwy` (done-with-you: shared), or `diy` (do-it-yourself: the client runs it).
     Ask which fits; default to the agency's usual if they have one.
   - **swarm?** — if this is a multi-client "swarm" workspace, set `is_swarm: true`
     and ask for `swarm_seat_limit` (required when swarm) and optionally
     `swarm_client_types`.
   - **data_region** — `ca` / `eu` / etc. Leave null to inherit from the agency
     (the common case).

3. **Confirm, then create.** Echo back the exact settings ("Create workspace
   *Acme Co* — done-for-you, region inherited, active") and get a yes. Then call
   `create_sub_account` with those args. The MCP's confirm-before-write gate may
   return a confirmation envelope first — relay it and re-confirm.

4. **Set them up for success.** After creation, offer the obvious next steps in
   the new workspace: connect an Instagram/channel account, create the first
   client (`create_client`), or build a starter workflow (hand off to
   **build-workflow**).

## Output

- Confirmation of the created workspace (name + id + settings).
- A short "next steps in this workspace" list.

## Guardrails
- Always confirm the name and operating mode before creating — a workspace is a
  real tenant, not a throwaway.
- `swarm_seat_limit` is required if `is_swarm` is true; don't create a swarm
  workspace without it.
- If `check_connection` shows a plan tier that can't manage workspaces, say so
  and stop rather than attempting the call.
