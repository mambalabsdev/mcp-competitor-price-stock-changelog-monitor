# Competitor Price, Stock, and Changelog Monitor MCP Server

[![Smithery](https://smithery.ai/badge/mambabuilt/mcp-competitor-price-stock-changelog-monitor)](https://smithery.ai/servers/mambabuilt/mcp-competitor-price-stock-changelog-monitor) [![Glama score](https://glama.ai/mcp/servers/mambalabsdev/mcp-competitor-price-stock-changelog-monitor/badges/score.svg)](https://glama.ai/mcp/servers/mambalabsdev/mcp-competitor-price-stock-changelog-monitor) [![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%3Fsearch%3Dcom.mambabuilt%252Fmcp-competitor-price-stock-changelog-monitor%26limit%3D1&query=%24.servers%5B0%5D._meta%5B%22io.modelcontextprotocol.registry%2Fofficial%22%5D.status&label=mcp%20registry&color=blue)](https://registry.modelcontextprotocol.io/v0/servers?search=com.mambabuilt/mcp-competitor-price-stock-changelog-monitor&limit=1) [![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-competitor-price-stock-changelog-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-competitor-price-stock-changelog-monitor) [![npm downloads](https://img.shields.io/npm/dm/@mambalabsdev/mcp-competitor-price-stock-changelog-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-competitor-price-stock-changelog-monitor) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-competitor-price-stock-changelog-monitor)](https://github.com/mambalabsdev/mcp-competitor-price-stock-changelog-monitor/blob/main/LICENSE) [![mcpservers.org](https://img.shields.io/badge/mcpservers.org-listed-blue)](https://mcpservers.org/servers/mambalabsdev/mcp-competitor-price-stock-changelog-monitor)

An MCP server for the Mamba Labs Competitor Price, Stock, and Changelog Monitor actor on Apify. Give it competitor sites. Each run compares every site with the state saved by the last run and returns only what changed.

## What's Inside

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Prerequisites](#prerequisites)
- [Example prompts](#example-prompts)
- [Tool](#tool)
- [Inputs](#inputs)
- [Pricing](#pricing)
- [Your keys](#your-keys)
- [Full actor documentation](#full-actor-documentation)
- [License](#license)

## What it does

- **Stores**: Shopify stores through their public product feed and WooCommerce stores through their public Store API. Changes: `price_change`, `compare_at_price_change`, `out_of_stock`, `back_in_stock`, `new_product`, `removed_product`, each with the product, variant, old and new value, and a link.
- **SaaS sites**: the pricing page and changelog the site links to. Changes: `plan_added`, `plan_removed`, `plan_price_change`, `pricing_page_change`, and `changelog_entry`.
- **One row per site per run**: `site_baseline` on the first check under a `state_key`, then `site_check` with `changes_found` (`no_change` when nothing moved).

Unreadable sites come back free with a `row_status` saying so.

All of the work runs on Apify. This package is a thin client that starts the actor run, waits for it, and hands back the dataset rows.

## Quick start

You need Node.js 18 or newer and an Apify account with an API token.

Add this to your Claude Desktop config (`claude_desktop_config.json`), Cursor, or any MCP client:

```json
{
  "mcpServers": {
    "mamba-competitor-price-stock-changelog-monitor": {
      "command": "npx",
      "args": [
        "-y",
        "@mambalabsdev/mcp-competitor-price-stock-changelog-monitor"
      ],
      "env": {
        "APIFY_TOKEN": "your-apify-token"
      }
    }
  }
}
```

Restart the client. The `monitor_competitor_changes` tool appears.

## Prerequisites

- Node.js 18 or newer.
- An Apify API token from [console.apify.com/account/integrations](https://console.apify.com/account/integrations), set as `APIFY_TOKEN`. The server lists its tool without a token; a call needs one.

## Example prompts

- "Watch deathwishcoffee.com and allbirds.com for price cuts of 10 percent or more and tell me what changed since last week."
- "Did Notion or Linear change their pricing plans or ship anything new this week?"
- "Which products went out of stock on gymshark.com since yesterday?"

## Tool

`monitor_competitor_changes`. One tool. Example arguments:

```json
{
  "sites": [
    "deathwishcoffee.com",
    "linear.app"
  ],
  "state_key": "competitors-weekly",
  "min_price_change_pct": "5"
}
```

## Inputs

Every buyer facing input of the actor, unchanged. Omitted inputs take the actor's own defaults. Numbers and booleans are sent as strings.

| Input | Type | Description |
|---|---|---|
| `sites` | array | Competitor sites to watch, one per line: a domain (deathwishcoffee.com, linear.app) or a URL. A URL with a path is read as that page: a pricing page, a changelog page, or a product page. Up to 200 sites per run |
| `site_type` | string | auto (default) reads the public Shopify product feed or the WooCommerce Store API when the site has one, and otherwise reads the pricing and changelog pages it links to |
| `watch` | array | Any of: prices, stock, products, plans, changelog. prices covers price and compare at price per variant; stock covers in stock and out of stock; products covers new and removed products; plans covers plan names and prices on a pricing page; changelog covers new changelog entries. Empty means all five |
| `product_urls` | array | Product pages on a store that is not Shopify or WooCommerce. Each page is read for its schema.org Product price and availability |
| `pricing_urls` | array | Pricing pages to read, when the site does not link to its pricing page from the homepage |
| `changelog_urls` | array | Changelog or release notes pages to read, when the site does not link to one from the homepage. An RSS or Atom feed the page declares is read first |
| `max_products` | string | Products read from a store catalog per run, up to 5000. A store with more products than this is compared on the products read, and removed products are not reported for it. Default 250 |
| `min_price_change_pct` | string | Report a price change only when it is at least this many percent. 0 reports every change. Default 0 |
| `max_changes_per_site` | string | Change rows returned per site per run, up to 1000. The rest are counted in changes_found. Default 200 |
| `include_snapshot` | string | true to also return one snapshot row per product variant, plan, and changelog entry on the first check of a site. Each snapshot row is billed as a snapshot row. Default false |
| `state_key` | string | Names the saved state, so separate schedules keep separate baselines. Runs with the same state key and site compare against each other |
| `reset_state` | string | true to ignore the saved state and record a new baseline for every site. Default false |
| `max_charge_usd` | string | Optional. The run stops before it would charge more than this |
| `proxy_mode` | string | auto (default) sends requests directly and moves a host to the Apify proxy default pool only when that host refuses. A reliability setting: proxy use adds nothing to your price |

## Pricing

Pay per event on Apify, billed to your Apify account. Prices in USD.

| Event | Free plan | Bronze | Silver | Gold |
|---|---|---|---|---|
| `change-row`, one change on a watched site since the last run: a price, compare at price, or stock change on a variant, a new or removed product, a plan added, removed, or repriced, a price change on a pricing page, or a new changelog entry | $0.0025 | $0.002375 | $0.00225 | $0.002125 |
| `site-check`, one site read and compared with its saved state, once per site per run | $0.003 | $0.00285 | $0.0027 | $0.00255 |
| `snapshot-row`, one current variant, plan, or changelog entry on the first check of a site, returned only when include_snapshot is on | $0.001 | $0.00095 | $0.0009 | $0.00085 |
| `apify-actor-start`, per run start | $0.0005 | $0.0005 | $0.0005 | $0.0005 |

Sites that cannot be read are free. A site checked with no changes costs one site check.

## Your keys

The actor needs no key. It reads public product feeds and public pages and holds no key of any kind. Proxy traffic adds nothing to your price: Mamba Labs pays the platform costs of the run. `proxy_mode` is a reliability setting.

## Full actor documentation

[apify.com/mambalabs/competitor-price-stock-changelog-monitor](https://apify.com/mambalabs/competitor-price-stock-changelog-monitor)

## License

MIT. See [LICENSE](LICENSE).
