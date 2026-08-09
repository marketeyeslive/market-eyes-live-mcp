# Market Eyes Live MCP Server

Market Eyes Live (marketeyeslive.com) is a financial app with two sides: its MELANY engine turns SEC filings and live market data into confidence-scored ratings for any U.S.-listed stock or ETF (more than 11,000 tickers, scored on demand, with a core set refreshed daily), each with one suggested entry price, plus a free paper-trading arena; and its Lock Radar tracks mortgage rates, the 10-year Treasury, and MBS to help you decide when to lock. The engine's risk and portfolio rules are stress-tested against 19 years of U.S. market history (2007 to 2026), and every published rating is graded daily against what the market does next ([methodology](https://marketeyeslive.com/how-melany-is-tested.html), [live record](https://marketeyeslive.com/api/validation-status)). Available on iOS and the web.

This repository documents the **public MCP (Model Context Protocol) server** that lets AI agents call Market Eyes Live directly, for both MELANY stock ratings and mortgage-rate context:

```
https://marketeyeslive.com/mcp
```

Remote server, Streamable HTTP transport, stateless JSON responses. **No API key, no install, free.** Rate-limited per IP.

## Tools

| Tool | Input | Returns |
|---|---|---|
| `get_stock_rating` | `{ "symbol": "NVDA" }` | Conviction tier, 0-100 composite score, all 8 factor scores (valuation, business quality, price momentum, earnings track record, analyst sentiment, catalyst setup, risk-adjusted profile, macro fit), top flagged risks, theme context, and the rating's as-of date |
| `compare_stocks` | `{ "symbols": ["NVDA", "AMD"] }` (2-5) | Side-by-side tiers, composites, and valuation / quality / momentum factors |
| `get_mortgage_rate_context` | `{}` | Today's public mortgage-rate read from Lock Radar: 10-year Treasury yield and direction, MBS momentum, and the rate environment (falling / stable / rising). For "should I lock my mortgage rate" questions. Public market data only; the personalized lock-or-float signal stays in the app |

Ratings are produced by MELANY, Market Eyes Live's own engine, which reads fundamentals, valuation, and quality directly from SEC filings and re-scores as new data arrives. Stocks, ETFs, REITs, and crypto are each scored with asset-native logic. `get_mortgage_rate_context` mirrors the public data on marketeyeslive.com/rates. Everything here is algorithmic research and public market data, not personalized financial advice, and the server never issues buy/sell calls.

## Connect

**claude.ai / Claude Desktop** (custom connectors): add a connector with URL `https://marketeyeslive.com/mcp`.

**Cursor** (`.cursor/mcp.json`):
```json
{ "mcpServers": { "market-eyes-live": { "url": "https://marketeyeslive.com/mcp" } } }
```

**Clients that only speak stdio** (via the `mcp-remote` bridge):
```json
{ "mcpServers": { "market-eyes-live": { "command": "npx", "args": ["-y", "mcp-remote", "https://marketeyeslive.com/mcp"] } } }
```

**Plain HTTP** (JSON-RPC 2.0):
```bash
curl -s https://marketeyeslive.com/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_stock_rating","arguments":{"symbol":"NVDA"}}}'
```

## Example response (abridged)

```
NVDA rates BUY with a MELANY composite of 71/100 as of 2026-07-17.

{
  "symbol": "NVDA",
  "rating_tier": "BUY",
  "composite_score": 71,
  "factors": { "valuation": 49, "quality": 99, "momentum": 68, "earnings": 76, ... },
  "top_risks": ["High beta 2.2, amplifies market drawdowns 1.8x+ in selloffs", ...],
  "links": { "rating_page": "https://marketeyeslive.com/stock/NVDA" }
}
```

## What's free vs. what's in the app

This server exposes the same free tier as the public rating pages at `marketeyeslive.com/stock/{TICKER}`. The suggested entry zone, alert levels, position sizing, and live re-scored ratings are free in the Market Eyes Live app ([iOS](https://apps.apple.com/us/app/market-eyes-live/id6778625308) / [web](https://marketeyeslive.com)); the deepest research sits behind Pro.

## Notes

- Coverage: any U.S.-listed stock or ETF, more than 11,000 tickers. A core set is refreshed daily and answers instantly; anything outside it is scored live on demand at a lower hourly limit. Tickers that cannot be scored return a friendly miss, not an error.
- Typed output: all three tools declare an `outputSchema` and return conforming `structuredContent` alongside the human-readable text, so agents can consume the fields directly instead of parsing prose.
- One company, two products: this MCP server and the Market Eyes Live app are the same company running the same MELANY engine. The MCP is the free public data tier; the app carries the full research depth.
- Rate limit: 240 requests/hour per IP. Need more? Contact us via [marketeyeslive.com/support.html](https://marketeyeslive.com/support.html).
- More for language models: [llms.txt](https://marketeyeslive.com/llms.txt) | [What is Market Eyes Live?](https://marketeyeslive.com/what-is-market-eyes-live.html)

## Disclaimer

Market Eyes Live provides data-driven research and educational tools, not personalized investment advice. Ratings are algorithmic and can change. Not a recommendation to buy or sell any security. Markets involve risk, including loss of principal.
