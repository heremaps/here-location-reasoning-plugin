---
inclusion: always
---

# HERE Location Reasoning Plugin — capability routing

This workspace has the **HERE Location Reasoning plugin** installed. It bundles two HERE capability sets, each backed by an MCP server and a skill. This file is the always-on router: it tells you *which* capability a request belongs to and how to reach the skill that carries the detail. The two skills hold the depth — load and follow the relevant one.

## The two capabilities

- **Doing a location operation** → the HERE Location Reasoning (HLR) MCP tools (geocoding, place discovery, routing, transit, isolines, traffic, matrix, map visualization). Before acting, load the **`location-actions`** skill for tool choice, chaining, and the failure modes the tool schemas don't warn about — read it with the `readSkill` tool (skill `location-actions` in power `here-location-reasoning-plugin`). Its guidance is easy to get wrong by guessing, so apply it rather than rediscovering it.
- **A question about HERE** (which product/API, how something works, setup, auth, pricing, integrating HERE into your own code) → the HERE Docs MCP (`search`/`fetch` plus the OpenAPI tools). Before answering, load the **`here-docs`** skill for how to query the docs, the intent-to-slug catalog, and product disambiguation — read it with the `readSkill` tool (skill `here-docs` in power `here-location-reasoning-plugin`).

## Routing rules

1. **Answer "about HERE" questions from the HERE Docs MCP, not a web search.** For any question about a HERE product, API, concept, setup, authentication, pricing, or how to connect/integrate a HERE service, call the HERE Docs MCP (`search`, then `fetch` the best hit) before anything else. The HERE Docs MCP is the authoritative, current source; web search is stale and often wrong on HERE specifics. Only fall back to the web if the docs genuinely lack the answer, and say so when you do.

2. **"Connecting HERE" is a docs question — even though this plugin already wires HLR into Kiro.** The plugin gives *this* Kiro session the HLR tools. That does **not** answer a user asking how to connect or integrate a HERE service (including HLR) in their *own* or *external* project — a different IDE, a Python/TypeScript app, or an agent framework such as PydanticAI, LangChain, or Strands. Treat those as docs questions and retrieve the answer from the HERE Docs MCP (the `location-reasoning` docs include ready-to-run integration examples). Do not dismiss them with "you already have it connected."

3. **To perform a location task, call the HLR tools — don't just describe them.** If the user wants directions, places, reachability, travel-time comparisons, traffic, or a map, run the tools and give the result.

4. **Don't call HLR tools to answer a question *about* HLR.** Questions about HLR's setup, tools, or connectivity are documentation lookups (HERE Docs MCP), not location operations. Never invoke HLR tools recursively to describe HLR.

5. **Don't invent HERE facts.** Product names, doc slugs, endpoints, parameters, and tool arguments come from the HERE Docs MCP or the tools' own schemas — never from memory or guesswork. A plausible-but-wrong HERE detail is worse than looking it up.
