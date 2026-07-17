# Inflowave Agent Skills

> Official Agent Skills for **[Inflowave](https://inflowave.io)** — the Instagram-automation
> and lead/sales platform. [Website](https://inflowave.io) ·
> [MCP docs](https://inflowave.io/mcp-documentation) · [App](https://app.inflowave.io)

Curated **Agent Skills** that turn the [Inflowave MCP](https://inflowave.io/mcp-documentation)
(292 tools) into a handful of plain-English superpowers. Instead of asking your AI
to remember which of 292 tools to call, install these skills and just say
*"run my weekly recap"* or *"find what's broken in my workflows and fix it."*

Every skill is a playbook over the live [Inflowave MCP](https://mcp.inflowave.io/mcp)
— read-first, correlate across domains (workflows · forms · analytics · pipeline),
and end with a **prioritized, do-this-next** suggestion list.

Don't have Inflowave yet? [Start here → inflowave.io](https://inflowave.io)

## Install

```bash
# One-liner (installs the whole hub)
npx skills add inflowave/skills

# …or a single skill
npx skills add inflowave/skills/growth-radar
```

You need the Inflowave MCP connected first. The fastest path:

```bash
npm i -g @inflowave/cli
inflowave auth login        # paste your key from Settings → API
inflowave mcp add           # wires it into Claude / Cursor / any MCP client
```

(or add it by hand — `claude mcp add --transport http inflowave https://mcp.inflowave.io/mcp --header "Authorization: Bearer YOUR_API_KEY"`)

## Skills

| Skill | What you say | What it does |
|---|---|---|
| **weekly-recap** | *"run my weekly recap"* | A narrated business review — growth, pipeline, revenue, top workflows — ending in 3 prioritized moves. |
| **workflow-doctor** | *"audit my workflows and fix them"* | Finds broken / underperforming / stalled workflows, correlates failures to the exact node, proposes and (on OK) applies fixes, then test-runs them. |
| **form-insights** | *"how are my forms doing?"* | Reads submissions, correlates form → lead → pipeline conversion, flags leaks (submitted-but-no-follow-up), and offers to build the follow-up workflow. |
| **build-workflow** | *"build a nurture sequence for new leads"* | Guided, valid-by-construction workflow build (trigger → actions → edges), then a safe test run before you activate. |
| **growth-radar** | *"where's my biggest opportunity?"* | Cross-domain correlation engine — which sources/forms/workflows actually drive revenue — surfaced as a ranked opportunity list with the exact next action. |
| **create-workspace** | *"spin up a new workspace for Acme"* | Creates a sub-account (workspace) with the right operating mode and optional swarm seats, confirming the name first. |

## Design principles (every skill follows these)

1. **Read before write.** Pull the data, show the finding, and get an explicit
   yes before any create/update/delete.
2. **Correlate, don't just list.** A number alone isn't insight — always tie it
   to a cause (which node, which source, which form) and a suggested action.
3. **Confirm the workspace.** Inflowave is multi-tenant. Resolve the sub-account
   / client by name and echo it back before writing (the MCP enforces this too).
4. **Plain language out.** No tool names, no jargon, no p95s — write for a
   business owner. End with a short, ranked "do this next" list.
5. **Test writes.** For workflows, offer `test_workflow` (simulated — nothing is
   sent to real people) before activating anything.

## Learn more

- **Website** — https://inflowave.io
- **MCP documentation** — https://inflowave.io/mcp-documentation
- **Connect Claude / Cursor / any AI** — https://inflowave.io/mcp-documentation
- **App / generate an API key** — https://app.inflowave.io/settings?tab=api

---
Built and maintained by **[Inflowave](https://inflowave.io)** — Instagram automation +
lead & sales, run from plain language. Tools referenced here are live as of 2026-07.
[inflowave.io](https://inflowave.io)
