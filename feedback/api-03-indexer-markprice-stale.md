# Indexer `Market.markPrice` lags the pool mark by 19–44 minutes on every market

## Severity
major

## Area
API

## Summary
The GraphQL indexer's `Market.markPrice` (the field the SDK surfaces as
`PerpMarket.markPrice` from `client.listPerpMarkets()`) is stale by
19–44 minutes on all 21 perp markets, deviating from the live pool mark by
17–138 bps at measurement time — while the same rows' `lastTradeAt` shows the
markets traded seconds earlier. Other indexed state (open interest, funding
rate, cumulative funding, order-book parameters) tracks the chain exactly, so
this is specific to how the market row's mark is refreshed: any consumer
pricing margin, liquidation distance, or PnL off the documented indexer field
is working from a materially old price with no staleness warning in the docs.

## Steps to reproduce
1. Query the indexer:
   ```graphql
   { Market(where: {perpRegistered: {_eq: true}}) {
       baseSymbol markPrice markPriceUpdatedAt lastTradeAt } }
   ```
   Endpoint: `https://hideki-dev.smk.somnia.host/v1/graphql`
2. At the same moment, read the live mark from each pool:
   `PerpPool(poolAddress).tryGetMarkPrice()` (or `getMarkPrice()`) via
   `https://api.hideki2.infra.testnet.somnia.network`.
3. Compare `markPriceUpdatedAt` against now, and the two prices.

Measured 2026-10-20 ~06:18 UTC / 13:18 WIB (epoch 1792474703):

| Market | indexer mark age | last trade age | |indexer − chain| |
|---|---|---|---|
| ETH | 1,146 s | 4 s | 32 bps |
| BTC | 1,600 s | 9 s | 22 bps |
| LINK | 1,424 s | 54 s | 41 bps |
| SOL | 2,652 s | 24 s | 59 bps |
| WLD | 1,423 s | 27 s | 95 bps |
| PUMP | 2,584 s | 18 s | 96 bps |
| ZEC | 2,585 s | 21 s | 90 bps |
| VVV | 2,582 s | 27 s | **138 bps** |
| …(all 21 markets) | 1,146–2,652 s | mostly < 60 s | 16–138 bps |

The ages cluster (≈1,420 s for many markets, ≈2,580 s for others), consistent
with batch sweeps on a slow cadence rather than per-trade updates. A concrete
point sample: the BTC row's `markPrice` was byte-identical
(81918114202770129000000, `markPriceUpdatedAt = 1792473106`) in an SDK
`listPerpMarkets()` read and in a GraphQL read ~50 minutes later, while the
on-chain mark moved from ≈81918 to 81731.94 over the same session.

## Expected
`Market.markPrice` tracks the pool's mark within a few seconds / a few bps
(like the indexer's openInterest and funding fields do), or the field is
documented as a slow-cadence value.

## Actual
20–45-minute-old marks on actively trading markets, deviating up to ~1.4%,
with no documentation of the cadence.

## Evidence
- Raw samples: `evidence/api/mark-staleness.json` (all 21 markets: on-chain
  mark, indexer mark, ages, deviation in bps), `evidence/api/marks-indexer-now.json`,
  `evidence/api/funding-onchain.json` / `funding-indexer.json` (funding fields
  shown to match the chain exactly at the same time).
- Live cross-check commands: GraphQL query above; `cast call <pool>
  "getMarkPrice()(uint256)"` on the Hideki RPC.

## Environment
- Network: Somnia Hideki testnet, chain ID 50383
- Indexer: `https://hideki-dev.smk.somnia.host/v1/graphql`
- SDK: `@somnia-chain/markets-sdk` 0.32.1 (`client.listPerpMarkets()` returns
  this field as `markPrice`)
