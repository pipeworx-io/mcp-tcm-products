# @pipeworx/tcm-products

Traditional Chinese Medicine (TCM) / Chinese Proprietary Medicine products
licensed in Singapore — the Health Sciences Authority (HSA) "Listing of
Licensed Chinese Proprietary Medicine Products" (~12,000 products), sourced
from data.gov.sg, searchable by Chinese characters, pinyin (spaced or not),
or English name.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

An HSA licence is a **safety and quality listing only** — it is not evidence
that a product treats any condition. The dataset carries no ingredients and
no indications; it is a product inventory, and the tools describe it as one.

## Tools

- `tcm_product_search(query?, company?, manufacturer?, country?, dosage_form?, limit?, offset?)` —
  products matching a name in any script (板蓝根 / banlangen / Ban Lan Gen /
  SANJIN all work; the licensed name fields carry Chinese characters inline
  next to the romanised names, so matching needs no translation source).
  Filters are case-insensitive substring matches on their columns.
- `tcm_product_manufacturers(country?, manufacturer?, limit?)` — manufacturers
  aggregated with licensed-product counts and countries of manufacture; the
  supply-chain view of who makes licensed CPM products and where.

## Why a separate pack, not tools on data-gov-sg

Fleet #1380. `data_gov_sg_query_dataset` filters by exact column value, so it
structurally cannot answer an alias query, and `search_packs("traditional
chinese medicine")` found nothing TCM-shaped — the gap was discoverability.
A pack whose name and descriptions say "Traditional Chinese Medicine" is what
makes the catalog search find the data; two named tools buried in a
general-purpose Singapore pack would not.

## Auth

Keyless.

## Data sources

- <https://data.gov.sg/api/action/datastore_search?resource_id=d_2ae2e6beb458d059318c1c14ad899f98> —
  the full export. One request returns all rows (`limit=12000` → 11,977 rows,
  ~1.9 MB, verified 2026-09-08); the pack fetches it once per isolate and
  caches for 24h, because data.gov.sg rate-limits aggressively per caller
  (429s measured in fleet #1380) and the dataset updates roughly monthly.
- CKAN's full-text `q` parameter is **rejected** by data.gov.sg
  (`"q is invalid"`, verified 2026-09-08) — server-side text search does not
  exist on this surface, which is why matching is done in the Worker.
- Dataset metadata: <https://api-production.data.gov.sg/v2/public/api/datasets/d_2ae2e6beb458d059318c1c14ad899f98/metadata>
  (managed by Health Sciences Authority, last updated 2026-08-07 at check time).

## Licence

Singapore Open Data Licence v1.0 (<https://data.gov.sg/open-data-licence>) —
worldwide, perpetual, royalty-free, commercial use permitted with attribution.
Verified 2026-09-08: the dataset page shows "Open Data Licence" and the licence
text covers datasets on data.gov.sg. Attribution notice (included in every tool
response): *Contains information from "Listing of Licensed Chinese Proprietary
Medicine Products" (Health Sciences Authority) from data.gov.sg, made available
under the terms of the Singapore Open Data Licence version 1.0.*

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "tcm-products": {
      "url": "https://gateway.pipeworx.io/tcm-products/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/tcm-products/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/tcm_product_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"板蓝根"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/tcm_product_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "tcm-products": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-tcm-products"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-tcm-products
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Tcm Products data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
