---
name: Read DerivaDEX market data
description: >-
  Fetch live perpetual-swap market data from the DerivaDEX public REST API —
  discover tradable symbols, read the L3 order book, tickers, and mark prices —
  with no authentication required.
api: openapi/derivadex-exchange-openapi.yml
operations: [listSymbols, getExchangeInfo, getTickers, getOrderBook, getMarkPrices, ping]
---

# Read DerivaDEX market data

DerivaDEX is a decentralized perpetual-swap exchange. The public REST market-data
surface at `https://exchange.derivadex.com` requires **no authentication**. Every
response uses the envelope `{ "value": ..., "timestamp": <unix>, "success": <bool> }`;
treat `success: false` as an error and read the payload from `value`.

## Steps

1. **Confirm connectivity** — `ping` (`GET /exchange/api/v1/ping`). Expect
   `success: true`.
2. **Discover products** — `listSymbols` (`GET /exchange/api/v1/symbols`) for the
   tradable perpetuals (e.g. `BTCP`, `ETHP`, `DDXP`), or `getExchangeInfo`
   (`GET /exchange/api/v1/exchange_info`) for tick sizes, min order size, and
   settlement epochs.
3. **Read tickers** — `getTickers` (`GET /exchange/api/v1/tickers?symbol=BTCP`)
   for last/mark/index price, funding rate, and open interest. Prices are
   decimal **strings** — parse with a decimal type, never a float.
4. **Read the order book** — `getOrderBook`
   (`GET /exchange/api/v1/order_book?symbol=BTCP&depth=10`). `side` is `0`=bid,
   `1`=ask.
5. **Read mark-price history** — `getMarkPrices`
   (`GET /exchange/api/v1/mark_prices?symbol=BTCP&limit=100&order=desc`). Page
   with the `globalOrdinal` cursor.

## Rules

- No strict rate limits are enforced, but excessive polling "may result in
  temporary restrictions" — back off on repeated calls.
- All prices/sizes/amounts are strings for precision; addresses and hashes are
  hex strings.
- This skill is read-only. Placing orders requires the Authenticated REST API and
  an EIP-712 wallet signature, which is not part of the public surface captured
  here (see `authentication/derivadex-authentication.yml`).
