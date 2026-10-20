# SDK `PERPS.md` market coverage is stale: claims only BTC/ETH perps and its WBTC example returns an empty list (21 perp markets are live)

- **Severity:** normal
- **Area:** SDK
- **Environment:** `@somnia-chain/markets-sdk` v0.32.1 (`docs/PERPS.md` inside the package), Somnia Hideki testnet, chain 50383, indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, read-only

## Summary

Two statements in the shipped `docs/PERPS.md` do not match the live testnet the doc describes:

1. Line 3: "Perps are live on testnet: `BTC/USDSO:USDSO` and `ETH/USDSO:USDSO`" — in fact **21 perp markets** are live on Hideki (1000PEPE, ADA, AVAX, BNB, BTC, DOGE, ETH, HYPE, LINK, LIT, NEAR, PUMP, SOL, SUI, TAO, VVV, WLD, XLM, XRP, ZEC, ZRO), as returned by the SDK's own `client.listPerpMarkets()`.
2. The "One-shot reads" example filters by a market that does not exist as a perp, so the example crashes on the next line:

```ts
const perps = await client.listPerpMarkets({ baseSymbol: "WBTC", limit: 20 });
const one = await client.getPerpMarket(perps[0].id); // null if not a perp
```

`WBTC` is a spot-only asset on Hideki — there is no WBTC perp market — so `listPerpMarkets({ baseSymbol: "WBTC" })` returns `[]` and `perps[0].id` throws.

## Steps to reproduce

```js
const perps = await markets.client.listPerpMarkets();
console.log(perps.length, perps.map(m => m.baseSymbol));

const wbtc = await markets.client.listPerpMarkets({ baseSymbol: "WBTC", limit: 20 });
console.log(wbtc);        // -> []
console.log(wbtc[0].id);  // -> TypeError
```

## Expected

Docs name the markets that exist (or use a real perp base symbol such as `BTC` in the example), so the example yields a non-empty list and `getPerpMarket` can be demonstrated.

## Actual

```
21 [ 'ZRO','LIT','VVV','1000PEPE','PUMP','ZEC','TAO','WLD','NEAR','XLM','AVAX','LINK','SUI','ADA','BNB','DOGE','XRP','HYPE','SOL','ETH','BTC' ]
[] 
TypeError: Cannot read properties of undefined (reading 'id')
```

## Evidence

- Full live market list (21 rows with symbols/decimals): `evidence/sdk/test-live-output.txt`
- WBTC example reproduction: `evidence/sdk/test-live8-output.txt`
