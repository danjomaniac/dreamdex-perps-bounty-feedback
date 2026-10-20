# SDK docs use wrong perp symbol casing (`USDSO`) — every unified perp example throws "unknown symbol"

- **Severity:** major
- **Area:** SDK
- **Environment:** `@somnia-chain/markets-sdk` v0.32.1 (npm, as shipped — `docs/PERPS.md`, `docs/EXCHANGE.md` inside the package), Somnia Hideki testnet, chain 50383, indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, read-only (no wallet needed)

## Summary

The SDK's own documentation writes every perp symbol with an all-caps quote currency, e.g. `BTC/USDSO:USDSO`. The symbols the SDK actually registers are `BTC/USDso:USDso` (the quote token's real symbol is `USDso`). Symbol lookup is exact-case, so **every** unified-API perp example in the shipped docs fails with `InvalidInputError: unknown symbol BTC/USDSO:USDSO — call loadMarkets() first` — even immediately after a successful `loadMarkets()`, which makes the error message itself misleading (the markets *are* loaded; the casing is wrong).

Occurrences in the installed package (v0.32.1):

- `docs/PERPS.md` line 3: "Perps are live on testnet: `BTC/USDSO:USDSO` and `ETH/USDSO:USDSO`"
- `docs/PERPS.md` Quick start (~line 1197-1200): `exchange.depositMargin("BTC/USDSO:USDSO", 1_000)`, `exchange.createOrder("BTC/USDSO:USDSO", "limit", "buy", 0.001, 62_000)`, `exchange.fetchFundingRate("BTC/USDSO:USDSO")`
- `docs/EXCHANGE.md` line 39: perp symbol table example `BTC/USDSO:USDSO`

## Steps to reproduce

```js
import { SomniaMarkets, SOMNIA_HIDEKI_TESTNET_ADDRESSES } from "@somnia-chain/markets-sdk";
import { hidekiTestnet } from "@somnia-chain/markets-sdk/chains";

const markets = new SomniaMarkets({
  chain: hidekiTestnet,
  indexerUrl: "https://hideki-dev.smk.somnia.host/v1/graphql",
  addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES,
});
await markets.loadMarkets();
console.log(markets.symbols.filter(s => s.includes("BTC/"))); // -> "BTC/USDso:USDso" exists

await markets.fetchTicker("BTC/USDSO:USDSO");      // docs casing
await markets.fetchFundingRate("BTC/USDSO:USDSO"); // docs casing
```

## Expected

The documented examples resolve the BTC perp market (or the SDK normalizes symbol casing).

## Actual

```
InvalidInputError: @somnia-chain/markets-sdk: unknown symbol BTC/USDSO:USDSO — call loadMarkets() first
```

Same failure for `fetchTicker`, `fetchFundingRate`, `depositMargin("BTC/USDSO:USDSO", …)` and `createOrder("BTC/USDSO:USDSO", …)` (all four reproduced live). The identical calls with the live casing succeed:

```
await markets.fetchFundingRate("BTC/USDso:USDso");
// -> { symbol: "BTC/USDso:USDso", markPrice: 81884.07…, fundingRate: 0.001743371176123083, … }
```

## Evidence

- Live run output: `evidence/sdk/test-live4-output.txt` (docs casing throws for `fetchTicker`/`fetchFundingRate`; live casing returns full ticker/funding), `evidence/sdk/test-live8-output.txt` (`depositMargin`/`createOrder` with docs casing throw the same error before any transaction).
- Live symbol list (21 perps, all `…/USDso:USDso`): `evidence/sdk/test-live4-output.txt`.
