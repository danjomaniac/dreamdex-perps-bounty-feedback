# Indexer `Order.pool` is null for every perp order

## Severity
normal

## Area
API

## Summary
The indexer's `Order` entity carries a `pool` column, but it is never
populated for perpetual orders: in a 200-row sample of perp orders it was set
on 0 rows (spot orders: 8 of 200). Sibling entities key everything by pool —
`PerpPosition.pool` (populated 50/50 in the same test), `PerpOrderRejection.pool`,
`PerpTwapSample.pool` — so a consumer filtering or joining orders by pool (the
natural query for an order book) silently gets nothing and must instead join
through `Order.market.poolAddress`.

## Steps to reproduce
GraphQL at `https://hideki-dev.smk.somnia.host/v1/graphql`:

```graphql
{ Order(where: {market: {marketType: {_eq: "PERP"}}}, limit: 200) { pool market_id } }
```

Count rows with a non-null `pool`. Repeat for spot orders and for
`PerpPosition(limit: 50) { pool }`.

## Expected vs actual
- Expected: `Order.pool` holds the pool address for perp orders, as it does on
  `PerpPosition`.
- Actual: `pool` is null on 200/200 perp orders sampled (and 192/200 spot
  orders). `market_id` is populated, and `market { poolAddress }` resolves, so
  the data exists in the indexer — it is just not written to `Order.pool`.

## Evidence
- Query results and counts: `evidence/api/pool-field-check.json`,
  `evidence/api/open-orders2.json` (individual rows showing `"pool": null`
  alongside a resolved market/poolAddress).

## Environment
- Network: Somnia Hideki testnet, chain ID 50383
- Indexer: `https://hideki-dev.smk.somnia.host/v1/graphql`
