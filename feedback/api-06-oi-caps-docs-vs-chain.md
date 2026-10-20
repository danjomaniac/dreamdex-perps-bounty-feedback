# Market risk numbers in the docs no longer match the chain: OI caps (BTC, NEAR) and the per-account position cap

## Severity
normal

## Area
docs

## Summary
The trader market table's "Open interest cap" column is presented as read from
the live markets. Checked against `getPerpPoolParameters()` on every pool, 19
of 21 caps match exactly — but two do not, and the text's position-cap rule
("A single account is capped at a tenth of it on most markets") fails in both
directions on flagship markets. This also resolves a related observation: the
BTC market's live open interest (274.99985 BTC) looks like it breaches the
documented cap (246.81545 BTC), but the real on-chain cap is 275 BTC — there
is no breach; the table is stale.

## Steps to reproduce
For each pool from the indexer market list, call
`PerpPool(pool).getPerpPoolParameters()` and `getOpenInterest()` on
`https://api.hideki2.infra.testnet.somnia.network`, and compare with
docs.dreamdex.io → Trading → Perps on Testnet → Markets.

## Divergences (raw base units, human units in brackets)
- **BTC-PERP OI cap**: docs 246.81545 BTC → on-chain `maxOpenInterest`
  27,500,000,000 [275 BTC]. Live OI 27,499,985,000 [274.99985 BTC] — under the
  real cap, over the documented one.
- **NEAR-PERP OI cap**: docs 250,410 NEAR → on-chain 279,000 NEAR.
- **Per-account cap (`maxPositionSize`) vs "a tenth of the cap"**:
  - BTC: 100 BTC against a 275 BTC market cap (≈36%, not 10% = 27.5).
  - ZEC: 420 ZEC against a 420 ZEC market cap — one account may hold the
    **entire** market cap; the documented "tenth" would be 42.
  - Eight markets (ADA, AVAX, BNB, DOGE, LINK, SUI, XLM, XRP) sit at
    ≈ one fifth (tighter than documented — harmless, noted for completeness),
    and NEAR at 50,000 of 279,000 (≈1/5.6). Only 11 of 21 markets are at the
    documented tenth.
- Related numeric claim: the Contracts → Functions page puts the builder-fee
  cap at 100,000 (100 bps) "on every pool", but the BTC PerpPool's
  `getMaxBuilderFeeBpsTimes1k()` returns 10,000 (10 bps = 0.1%) — a 10× lower
  cap on perps, and placement above it reverts `BuilderFeeExceedsCap`
  (0xf559e808).

## Expected vs actual
Expected: published caps and the "tenth" rule describe the deployed parameters
(the page says the figures "are read from the live markets").
Actual: stale caps for BTC/NEAR and a position-cap rule that materially
understates the allowed concentration on BTC and ZEC.

## Evidence
- Full 21-market comparison: `evidence/api/oi-cap-result.json`,
  `evidence/api/oi-cap-check.txt` (on-chain `getPerpPoolParameters` +
  `getOpenInterest` vs docs table vs indexer).
- Builder-fee cap read: `evidence/api/sim-builderfee.json`
  (`getMaxBuilderFeeBpsTimes1k` = "10000" on the BTC pool).

## Environment
- Network: Somnia Hideki testnet, chain ID 50383
- Contracts: per-market PerpPools (e.g. BTC `0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0`,
  NEAR `0x1cd15ae8bff0fe0a7f49233be8b90734775ba454`,
  ZEC `0x2a73129f26917124bd2a47a9b355c395c537f050`)
