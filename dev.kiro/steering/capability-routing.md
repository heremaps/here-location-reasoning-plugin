---
inclusion: always
---

# HERE Location Reasoning Plugin: capability routing

This workspace has the **HERE Location Reasoning plugin** installed: the HLR MCP (location tools) and the HERE Docs MCP (scoped to the `location-reasoning` docs).

- **To perform a location task** (directions, places, reachability, travel-time, traffic, a map), read the **`location-actions`** skill before calling the HLR tools. It carries the tool choice, chaining, and failure modes needed to use them correctly.
- **For any question about HERE Location Reasoning** (setup, auth, integration/connection, how something works), query the HERE Docs MCP (`search`/`fetch`) before anything else, including for connecting HLR into the user's own project or a different framework. It's the authoritative, current source, so prefer it over web search, and fall back to the web only if the docs genuinely lack the answer.
- **Don't call HLR tools to answer a question about HLR.** Documentation questions go to the Docs MCP, not to the location tools, even recursively.
- **Don't invent HERE facts.** Product details, endpoints, parameters, and tool arguments come from the Docs MCP or the tools' own schemas, never from memory.
