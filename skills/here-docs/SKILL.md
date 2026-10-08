---
name: here-docs
description: Find and read HERE documentation across all HERE products and API references through the HERE Docs MCP. Use this for questions ABOUT HERE - "which HERE API should I use", "how does HERE Routing v8 handle X", "how do I authenticate / set up the SDK", comparing HERE products, rate limits, API reference lookups, or any docs/how-to question. Resolves a query to the canonical HERE product slug, disambiguates similar product names, and drives the docs search/fetch and OpenAPI tools. Do NOT use this to DO a location operation (geocode, route, discover places, traffic, isolines, maps) — for performing a location task, use the location-actions skill.
---

# HERE Docs

Scopes a documentation query to the canonical HERE product slug and drives the HERE Docs MCP (`search`/`fetch` plus the OpenAPI tools) to find and read the right HERE docs.

## How to query the HERE Docs MCP

- `search` is **global and semantic**: one query ranks results across **all** HERE products in a single call, returning document IDs of the form `{slug}/{page-id}` plus titles and URLs.
- An ID prefixed with `ref:` is an API reference page; other IDs are the bare `{slug}/{page-id}` form.
- `fetch` retrieves a full document by its ID.
- OpenAPI tools for API-level detail: `list-specs`, `search-endpoints`, `get-endpoint`, `list-endpoints`. Their spec titles use the format `slug=SpecName` (e.g. `routing=Routing API v8`).

**Query strategy:** broad `search` → read the returned IDs to identify the product **slug** → `fetch` the best hit, or drill into API detail with `search-endpoints`/`get-endpoint`. No group index needs opening first, because search is already global.

## Finding the right product slug

Get slugs from the live sources, not from memory — a guessed slug misroutes silently.

- **Most queries need no lookup:** every `search` result ID is `{slug}/{page-id}`, so the slug is in the top hits. Read it off and `fetch` the best one.
- **For the full product list** (an "everything HERE offers" question, or a query whose product isn't obvious from the hits), fetch the site index at `https://docs.here.com/llms.txt`. It lists every project and links each project's own `{slug}/llms.txt`, and is the authoritative, always-current catalog.

## Disambiguation

For these pairs a generic query returns results from two similarly named products interleaved, so a top `search` hit alone won't disambiguate. When the query could mean either, draw the distinction below to decide *which product* you're after, then resolve its current slug from the `search` results.

| If the query could mean… | You want… | Not… | Because |
|--------------------------|-----------|------|---------|
| Traffic conditions **right now** | real-time traffic (Traffic API) | historical/analytics traffic | Analytics and historic-pattern data are aggregated over time, not live |
| EV **charge-point / connector data** | the EV charge-point dataset (EV Products) | EV-aware routing | Routing-EV plans range-aware routes; the dataset is the stations and connectors themselves |
| **Geospatial data catalogs & layers over HTTP** | the REST Data API | the Data SDK / pipeline workspace | The REST surface vs. the Java/Scala client libraries and pipelines for it |
| A **one-off device fix** (Wi-Fi/cell/GNSS) | Positioning | asset Tracking | Tracking is continuous asset location over time, not a single fix |
| **Static / raster map images** server-side | the raster/map-image rendering product | the browser JS maps library | The JS library renders interactive vector maps in a browser, not server-side images |
| Editing **map styles** in a browser tool | the no-code Style Editor | a code-based styling workflow | Style Editor is the browser tool for authoring cartographic styles |
| Docs **about** HERE Location Reasoning | the `location-reasoning` doc topic | the HLR tools | Documenting the topic is a docs task, not a location operation |

## Boundaries

- **Performing a location operation is not this skill's job.** Any request to actually geocode, calculate a route, discover places, get traffic, compute isolines, or render maps belongs to the `location-actions` skill. This skill is for documentation lookup, not for doing operations.
- **No recursive HLR.** HERE Location Reasoning appears here only as a documentation *topic* (slug `location-reasoning`). Answering docs questions about HLR — its setup, tools, or connectivity — is a docs task; it does **not** invoke the `location-actions` skill or call HLR tools recursively.
