# @pipeworx/census-tigerweb

The Census Bureau's TIGERweb ArcGIS REST service: every TIGER/Line geography
the Bureau publishes as a queryable layer — census tracts, block groups, 2020
blocks, ZCTAs, congressional and state legislative districts, school districts,
tribal areas, places, counties, urban areas and PUMAs, with their GEOIDs,
names, land/water area and interior points.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `tigerweb_list_layers(service?, filter?, include_label_layers?)` — the
  queryable layers in a TIGERweb service. Call this first; layer ids move
  between vintages.
- `tigerweb_query(layer_id, service?, where?, out_fields?, limit?, offset?, order_by?, return_geometry?)` —
  ArcGIS WHERE-clause query against any layer.
- `tigerweb_geography_at_point(latitude, longitude, service?, layers?)` — every
  geography containing a coordinate, in one call.

## Auth

Keyless.

## Data sources

- <https://tigerweb.geo.census.gov/arcgis/rest/services/TIGERweb/tigerWMS_Current/MapServer> — current-vintage geography (default).
- <https://tigerweb.geo.census.gov/arcgis/rest/services> — the service directory, including ACS and Census2020 vintages.

Companion to `census-geocoder`: the geocoder maps an address to a tract, this
pack lists and describes the geographies themselves.

## Traps

- Half of the 80 layers in `tigerWMS_Current` are `... Labels` layers — point
  annotation, not boundaries. Querying one returns rows that look like real
  geography. `tigerweb_list_layers` hides them unless asked.
- **Layer numbers move between services and vintages.** Never carry one over
  from another service.
- FIPS fields are zero-padded **strings**: `STATE='11'` matches, `STATE=11`
  does not. A wrong-typed WHERE clause returns zero rows with a 200.
- ArcGIS **errors** on an `outFields` name a layer does not have, and these
  layers do not share a schema (Census Tracts has no `BLOCK`, ZCTAs have no
  `COUNTY`). `tigerweb_geography_at_point` asks for `*` and trims afterwards.
- `resultRecordCount` caps a page; the service has its own max too. Ask for
  more and you silently get the service max.
- Geometry is large — `return_geometry` defaults to false.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "census-tigerweb": {
      "url": "https://gateway.pipeworx.io/census-tigerweb/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/census-tigerweb/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/tigerweb_list_layers \
  -H 'Content-Type: application/json' \
  -d '{"filter":"school"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/tigerweb_list_layers`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "census-tigerweb": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-census-tigerweb"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-census-tigerweb
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Census Tigerweb data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
