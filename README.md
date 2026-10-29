# @pipeworx/gie-storage

European natural gas storage and LNG terminal data from Gas Infrastructure Europe (GIE): how full EU gas storage is, by country, operator or site; LNG terminal inventory and send-out; the facility directory; and planned and unplanned outage reports.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `gie_gas_storage(country?, company?, facility?, date?, from?, to?, days?, _apiKey)`: daily gas storage level for the EU, one country, one operator or one facility. Covers fill % of working gas volume, gas in storage (TWh), injection and withdrawal (GWh/d), trend and the gas day. With no dates you get the latest gas day. Served with Pipeworx's platform GIE key; `_apiKey` overrides it.
- `gie_lng_terminals(country?, company?, facility?, date?, from?, to?, days?, _apiKey)`: daily LNG terminal inventory (1000 m³ LNG and GWh), send-out, and declared maximum inventory and send-out. Served with Pipeworx's platform GIE key; `_apiKey` overrides it.
- `gie_storage_facilities(dataset?, country?, query?, include_closed?)`: lists every storage system operator (or LNG operator) by country, with each facility's name, type, EIC code and operational dates. Use the EIC codes as `company` / `facility` in the other tools. Keyless.
- `gie_storage_outages(dataset?, country?, company?, facility?, type?, from?, to?, limit?, page?)`: planned and unplanned unavailability reports, newest first. Each report gives the facility, start and end, whether the end is confirmed or estimated, the capacity taken offline and the operator's reason. `from`/`to` filter on the publication date. Keyless.

## Auth

The two directory and outage tools are keyless. The two level tools need a GIE key (GIE refuses storage and LNG level calls without a personal API key; Pipeworx supplies its own, and a caller's `_apiKey` overrides it). Registration is free at https://agsi.gie.eu/account, and the same key works on both AGSI+ and ALSI+. Pass it as `_apiKey`. If no key is available at all, the tool refuses with "requires an API key" and points to the keyless tools.

## Data sources

- Gas Infrastructure Europe (GIE), AGSI+ Aggregated Gas Storage Inventory: daily underground gas storage levels and storage unavailability reports.
- Gas Infrastructure Europe (GIE), ALSI+ Aggregated LNG Storage Inventory: daily LNG terminal levels and terminal unavailability reports.

Reuse terms: GIE publishes this as public transparency-platform data, and the platform states no reuse licence (checked 2026-10-14). This pack fetches it live for each request and stores no copy, so attribute it to GIE AGSI+ / ALSI+. Every call goes live to GIE, and every response carries `source`, `licence` and `data_as_of`. For the level tools, `data_as_of` is the gas day GIE reports.

Notes for the next person:
- An outage filter on a facility is silently ignored by GIE unless the country and operator are also sent. `gie_storage_outages` resolves both from the directory when only `facility` is given.
- In the outage feed, `total` is the row count on the current page. `last_page` is the real page count.
- GIE returns an auth error as HTTP 200 with `{"error":"access denied","message":"Invalid or missing API key"}`. The pack reads the body and raises the error rather than returning an empty list.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "gie-storage": {
      "url": "https://gateway.pipeworx.io/gie-storage/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/gie-storage/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/gie_gas_storage \
  -H 'Content-Type: application/json' \
  -d '{"country":"DE"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/gie_gas_storage`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-gie-storage` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/gie-storage/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "gie-storage": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-gie-storage"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Gie Storage data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
