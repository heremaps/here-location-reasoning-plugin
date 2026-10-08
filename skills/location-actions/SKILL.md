---
name: location-actions
description: Perform HERE location operations with the HERE Location Reasoning MCP tools — geocoding, place/POI discovery, routing (car, truck, EV, public transit), travel-time and distance comparison, reachability/isolines, real-time traffic, and map visualization. Use this whenever a request asks you to DO a location task - directions, finding places, "how far / how long" between locations, "what can I reach in N minutes", trip or EV charging planning, getting somewhere by bus/train, checking traffic, or showing any of it on a map — even for a single location tool, and even if the user never says "route" or "map". It covers which tool to pick, how to chain them, how to reuse cached IDs, and the failure modes these tools have that their schemas don't warn you about. Do NOT use this for questions ABOUT HERE products, APIs, setup, pricing, or how-to — for "which HERE API", "how does X work", "how do I set up / authenticate", or any documentation lookup, use the here-docs skill instead.
---

# HERE Location Actions

You are driving a set of HERE location tools over MCP. This guide is about **which tool to use, in what order, and the traps the schemas don't tell you** — not about parameter details.

> **The live `tools/list` is authoritative.** Which tools exist, their exact parameters, required fields, and response shapes come from the running server's `tools/list` schema — always defer to it. This guide covers tool *selection* and *chaining* judgment, not the schema; where the two ever disagree, the schema wins. The tool set evolves, so treat a tool this guide doesn't mention as usable per its own schema, and a parameter detail here as a hint to confirm against it.

> For questions **about** HERE products, APIs, setup, or how-to (not doing an operation), use the **here-docs** skill — don't call location tools.

## Tools

These are the HERE location tools, mapped to the intent each one serves. Pick by intent, chain per the workflows below, and take exact parameters from each tool's own schema — this guide covers *which* tool and *when*, not argument-level detail.

| Intent | Tool |
|--------|------|
| Address / place name → coordinates | `geocode` |
| Coordinates → address | `reverse-geocode` |
| Find places by category or name (restaurants, EV charging, "coffee in Porto") | `discover` |
| Directions between two points (car, truck, EV, scooter, …) | `calculate-route` |
| Directions by bus / train / subway / tram / ferry | `public-transit-route` |
| Rebuild a route from an existing polyline / GPS trace | `get-route-from-polyline` |
| Evenly spaced stops along a route already calculated | `find-route-stops` |
| Compare / rank / measure **several** origin–destination pairs at once | `matrix-routing` |
| Area reachable within a time / distance / energy budget | `isoline-route` |
| Places *near* other places or a mapped feature ("coffee near EV charging") *(experimental)* | `find-places-near` |
| Places *far from* other places or a mapped feature ("hotels far from airports") *(experimental)* | `find-places-far-from` |
| Real-time traffic speed and congestion | `traffic-flow` |
| Draw results on an interactive map | `data-visualization` |

### Picking the routing tool

- **One origin, one destination, by road** → `calculate-route`.
- **By public transport** (the user says bus, train, metro, "by transit") → `public-transit-route`, not `calculate-route`.
- **More than one pair** — "how far is each of these from X", ranking, closest-of-many, many-to-many → **one** `matrix-routing` call. Do not loop `calculate-route`; that is slower, costlier, and easy to get inconsistent.
- **"What can I reach in N minutes / km / charge?"** → `isoline-route` (a reachable-area polygon), not a guessed circle.

## Rules that keep tool calls correct

1. **Coordinates are `{lat, lng}` objects, never `[lng, lat]` arrays.** The GeoJSON habit is backwards for these tools and fails silently or routes to the wrong place. Pass the `position` object a tool returned.

2. **Resolve a place to coordinates the right way.** To turn a *specific named place or address* into a coordinate, use `geocode`. To *find places by category or name*, use `discover` — it accepts a combined query like "pizza in Lisbon" or an `area`, so it often needs no separate geocode. Don't `geocode` a category word ("pharmacy"), and don't add a `geocode` step when `discover` already resolves the location.

3. **Reuse cached IDs; don't recompute geometry.** `calculate-route`, `isoline-route`, and `traffic-flow` return an id that references server-side geometry. Pass that id to follow-up tools instead of resending or recalculating — it's faster and keeps every step consistent.

4. **Only use data a tool returned.** Never invent coordinates, addresses, route ids, or place names. A fabricated value produces a plausible, wrong answer.

5. **Leave optional parameters unset unless the request needs them.** Extra constraints narrow results and trigger validation errors; add a parameter only when the user's ask requires it.

6. **Validate coordinates before sending them downstream:** `lat` in [-90, 90], `lng` in [-180, 180]. A swapped or out-of-range pair is the most common silent failure.

## Report route problems the tools flag

`calculate-route` (and `get-route-from-polyline`) return a `notices` array on each route. **Always tell the user about every notice whose class is `blocking`** — the route may be illegal or impassable for the chosen vehicle, and hiding it presents an undrivable route as usable. Mention `preferenceViolated` and `timeConditional` notices when they bear on the request. Don't rank notices by the tools' `severity` field — it doesn't track real impact.

For **toll cost**, read `routes[].summary.tolls`. Never add up `sections[].tolls`: one payment is repeated across the sections it covers, so summing over-reports it.

## Cached-ID reuse

Calculate once, then reference the id everywhere else in the chain.

| Produced by | Id | Accepted by |
|-------------|----|-------------|
| `calculate-route` / `get-route-from-polyline` | route id | `find-route-stops`, `discover` (`area.type=route`), `traffic-flow` (`in.type=route`), `data-visualization` |
| `isoline-route` | isoline id | `discover` (`area.type=isoline`), `data-visualization` |
| `traffic-flow` | traffic id | `data-visualization` |
| `public-transit-route` | transit route id | `data-visualization` |

## `discover` — scope the search to the right area

The `area` parameter decides *where* to look; the wrong type is a common mistake.

- **Near a point** → `{ type: "circle", center: {lat,lng}, radius: <m> }`
- **Along a route** ("on the way", "between A and B") → `{ type: "route", routes: [{ id: "<route-id>" }], width: <m> }` — use the cached route id, **not** a circle around the midpoint.
- **Inside a reachable area** ("what's within my 20-minute drive / my range") → `{ type: "isoline", id: "<isoline-id>" }` — not a guessed radius.
- **Map rectangle** → `{ type: "bbox", west, south, east, north }`
- **Country-wide** → `{ type: "country", countryCodes: [...], center: {lat,lng} }`

## Showing results on a map

When the user asks to *see*, *show*, *map*, or *visualize* results, a map is the natural answer — build one with `data-visualization` and present it **alongside** a short text summary, not instead of it. If no visual was requested, a clear text answer is enough; don't force a map.

Build the map from what you already have: pass cached route / isoline / traffic / transit-route ids directly, and add markers for geocoded points or POIs with meaningful captions (place names, not raw coordinates).

## Gotchas that cause wrong results or errors

- **EV routing needs a full vehicle profile, not just connector types.** The `ev` object requires a set of physical vehicle parameters (connector types plus drive and recuperation efficiency, drag, weight, …); a partial object is rejected. Read the `calculate-route` `ev` schema for the exact required fields rather than guessing — that's the authoritative list. With a valid profile, the response plans charging stops along the route.
- **`matrix-routing` is tier-limited, and the tier is set by the deployment.** Which modes and matrix sizes are allowed depends on the deployment's tier, and the live `matrix-routing` schema in `tools/list` reflects the active tier — treat it as the source of truth rather than assuming a mode. On a profile-only tier you must pass `profile` and `regionDefinition` (`{"type":"world"}`), let destinations default to the origins when measuring within one set, and omit `transportMode`, `routingMode`, `departureTime`, and `arrivalTime` (they're rejected); a higher tier additionally accepts flexible mode with those parameters. If a parameter you sent is rejected, the schema tells you which mode is active. Either way, results come back as flat row-major arrays: the value for origin *i* and destination *j* is at index `i * numberOfDestinations + j`, and a `null` entry means that pair couldn't be routed.
- **`find-route-stops` doesn't reroute for you.** The stops it returns lie along the *original* geometry. If the user wants to actually visit them, call `calculate-route` again with the stops as `via` waypoints — the first route doesn't pass through them.
- **Public transit needs a concrete date.** Resolve relative dates ("next Monday", "tomorrow") to an explicit date *with the year* before calling `public-transit-route`; a bare or year-less time gives wrong schedules.

## Workflows

Terse skeletons — adapt to the request, drop steps that don't apply.

**Route with a map**
1. `geocode` origin and destination.
2. `calculate-route` → keep the route id; surface any `blocking` notice.
3. `data-visualization` with the route id + origin/destination markers.

**POIs along a route**
1. `geocode` origin and destination → `calculate-route` (keep route id).
2. `find-route-stops` with the route id and a `query` for the POI.
3. If asked to see it: `data-visualization` with the route + POI markers.

**EV trip**
1. `geocode` origin and destination.
2. `calculate-route` with a complete `ev` profile (see the EV gotcha above).
3. Read the planned charging stops from the response; map them if asked.

**Reachability**
1. `geocode` the origin.
2. `isoline-route` with `rangeType` (time / distance / consumption) and `rangeValues`.
3. `data-visualization` with the isoline id — or feed it to `discover` (`area.type=isoline`) to find places inside it.

**Compare many routes**
1. Geocode any addresses into coordinates.
2. **One** `matrix-routing` call (origins, destinations, `profile`, `regionDefinition`).
3. Read off the closest / fastest, or rank by time or distance.

**Journey by public transport**
1. `geocode` origin and destination; resolve the date to an explicit `YYYY-MM-DD`.
2. `public-transit-route` (add `modes` / `departureTime` only if the user constrained them).
3. Summarize legs, transfers, and times; map with the transit route id if asked.

**Import an existing polyline**
1. `get-route-from-polyline` with the polyline → a route id (same shape as `calculate-route`).
2. Reuse that id for stops, traffic, discovery, or a map.

## When a call fails

Read the error — it usually names the offending parameter (a tool error can arrive even with a 200 response; it still means a parameter problem, so re-check against `tools/list`). Fix and retry. If a place is ambiguous or a value is missing, ask the user rather than guessing.
