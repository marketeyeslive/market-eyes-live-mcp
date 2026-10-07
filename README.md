# Market Eyes Live MCP Server

A free, read-only **Model Context Protocol (MCP) server** that returns Market Eyes Live's MELANY ratings for U.S.-listed stocks and ETFs (more than 11,000 tickers on its coverage list), plus a daily read of U.S. mortgage-rate conditions.

```
https://marketeyeslive.com/mcp
```

Remote server, Streamable HTTP transport, stateless JSON responses. **No API key, no install, free.** Full documentation: [marketeyeslive.com/mcp-server](https://marketeyeslive.com/mcp-server).

A rating is a conviction tier and a 0 to 100 composite score built from eight factor scores. Published ratings are recorded and graded daily against later market moves; the method and the record are public ([methodology](https://marketeyeslive.com/how-melany-is-tested.html), [raw feed](https://marketeyeslive.com/api/validation-status)).

## Tools

| Tool | Input | Returns |
|---|---|---|
| `get_stock_rating` | `{ "symbol": "NVDA" }` | Conviction tier (weakest to strongest: Unfavorable, Hold, Favorable, Highest Conviction; names without mature fundamentals carry Promising, Very Promising, Rising Star, Runner! or Catalyst Watch), 0 to 100 composite, the eight factor scores (valuation, quality, momentum, earnings, sentiment, catalyst, risk-adjusted, macro fit), flagged risks, theme context, as-of date |
| `compare_stocks` | `{ "symbols": ["NVDA", "AMD"] }` (2 to 5) | Side-by-side tiers, composites, and valuation, quality and momentum scores from the daily-refreshed set |
| `get_mortgage_rate_context` | `{}` | Today's public read: 10-year Treasury yield and direction, MBS momentum, the rate environment (falling, stable or rising), and what moves mortgage rates |

All three tools are read-only (`readOnlyHint: true`), declare an `outputSchema`, and return conforming `structuredContent` alongside the text. Every result carries `source` and a `disclaimer`; stock results also carry `links.rating_page` and `links.methodology`.

Coverage is **U.S.-listed stocks and ETFs only**. Crypto, futures, and non-U.S. listings (for example `SHOP.TO` or `BTC-USD`) return a "not covered" result. Five U.S. tickers that share their symbol with a futures contract or a coin (CORN, GOLD, WTI, BTC, ETH) are not rated, and the result says so. Class shares work in either spelling (`BRK-B` or `BRK.B`). A ticker that is not on the coverage list returns a result saying so. The tier is always one of: Unfavorable, Hold, Favorable, Highest Conviction, Promising, Very Promising, Rising Star, Runner!, Catalyst Watch. ETFs and REITs use the same ladder, chosen by the tone of the fund's own label in the app, and a leveraged or inverse fund never reads above Hold.

## Connect

**Claude Code**:
```
claude mcp add --transport http market-eyes-live https://marketeyeslive.com/mcp
```

**claude.ai and the Claude desktop app**: Settings, then Connectors, then add a custom connector with URL `https://marketeyeslive.com/mcp` and no authentication.

**ChatGPT**: Plugins, then **+**, then **Add custom MCP server**, URL `https://marketeyeslive.com/mcp`, authentication none (Plus, Pro, Business, Enterprise and Edu plans; some accounts need developer mode). Menu names may vary.

**Gemini CLI** (this repository is a Gemini CLI extension; `gemini-extension.json` points at the endpoint over Streamable HTTP):
```
gemini extensions install https://github.com/marketeyeslive/market-eyes-live-mcp
```

**Perplexity, Gemini, Grok, Mistral Le Chat**: see the [setup steps](https://marketeyeslive.com/mcp-server).

Set it up on a computer; once added, it also works in the phone apps of Claude and ChatGPT.

**Cursor** (`.cursor/mcp.json`):
```json
{ "mcpServers": { "market-eyes-live": { "url": "https://marketeyeslive.com/mcp" } } }
```

**Claude Desktop config file and VS Code**:
```json
{ "mcpServers": { "market-eyes-live": { "type": "http", "url": "https://marketeyeslive.com/mcp" } } }
```

**Plain HTTP** (JSON-RPC 2.0):
```bash
curl -s https://marketeyeslive.com/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_stock_rating","arguments":{"symbol":"AAPL"}}}'
```

## Example `structuredContent` (illustrative values)

```json
{
  "symbol": "AAPL",
  "rating_tier": "Highest Conviction",
  "composite_score": 72,
  "scale": "0-100, higher is stronger",
  "as_of": "2026-10-06",
  "factors": { "valuation": 41, "quality": 88, "momentum": 72, "earnings": 66, "sentiment": 58, "catalyst": 52, "risk_adjusted": 63, "macro_fit": 60 },
  "top_risks": ["Valuation stretched vs 5-year average"],
  "source": "Market Eyes Live",
  "disclaimer": "Algorithmic research data, not financial advice and not personalized to anyone. Market Eyes Live does not place trades.",
  "links": {
    "rating_page": "https://marketeyeslive.com/stock/AAPL?utm_source=mcp&utm_medium=agent&utm_campaign=AAPL",
    "methodology": "https://marketeyeslive.com/how-melany-is-tested.html?utm_source=mcp&utm_medium=agent&utm_campaign=methodology"
  }
}
```

## Limits

Only tool calls count; connecting, listing tools and pings are free.

- 240 tool calls per hour per user when the AI platform passes an anonymous user id (ChatGPT does), otherwise per network address.
- 3,000 per hour shared by a platform that passes no user id (for example Claude).
- 3,000 per hour shared by all callers that do not come through a platform.
- 6,000 per hour across all callers.
- Live scoring of a ticker outside the daily-refreshed set: 20 per hour per user or address, 100 per hour per platform, 150 per hour for callers that do not come through a platform, 300 per hour across all callers. Live scores are reused for 12 hours.

A limit hit returns an error result that names the limit and when it resets. For higher volume, email team@marketeyeslive.com. Terms: [marketeyeslive.com/api-terms](https://marketeyeslive.com/api-terms).

## Disclaimer

Market Eyes Live provides algorithmic research data and educational tools, not financial advice. Ratings can change and are not a recommendation to buy or sell any security. Markets involve risk, including loss of principal. The server is read-only and never places trades.
