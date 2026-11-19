# @pipeworx/mo-sos-business

Missouri Secretary of State business-entity register: search entities by name and read an entity by its charter number, live per request.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `mo_search_businesses(name, active_only?, limit?)` — entities whose name starts with the query: charter number (`file_number`), name, entity type, locale (domestic = formed in Missouri, foreign = formed elsewhere and registered in Missouri), status and creation date. Each row is graded `exact`, `legal_form_variant` or `name_prefix`; `hits` counts only the first two. The register returns the first 20 rows of a longer list; `total_in_register` says how many matched in all. Fictitious-name registrations are marked `name_kind: fictitious`; they are not companies.
- `mo_get_business(entity_number)` — one entity by charter number, e.g. `LC014419805`.

## Auth

Keyless.

## Data sources

- Missouri Secretary of State, business entity search. Every response carries `source` and `data_as_of` (the date of the live read).

Entity-level only. No individuals' names are served: the registered agent, which can be a person, is not returned. Inactive, dissolved and withdrawn entities are included unless `active_only` is set.

The register sits behind a bot-challenge layer. A challenge, an HTTP error or a page that is not the search form is reported as an error naming the status, never as an empty result.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mo-sos-business": {
      "url": "https://gateway.pipeworx.io/mo-sos-business/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mo-sos-business/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/mo_search_businesses \
  -H 'Content-Type: application/json' \
  -d '{"name":"EvlarSoft","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mo_search_businesses`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mo-sos-business` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mo-sos-business/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mo-sos-business": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mo-sos-business"]
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
ask_pipeworx({ question: "your question about Mo Sos Business data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
