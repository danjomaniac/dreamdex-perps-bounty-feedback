# Voucher market allowlist is not enforced: all 21 markets read "allowed", including the 6 the docs mark as blocked

## Severity
major

## Area
contracts

## Summary
The trader docs say the 1,000 USDso paper credit "can only open positions in
the markets on its allowlist", the market table marks six markets **No**
(1000PEPE, LIT, PUMP, VVV, ZEC, ZRO), and the developer guide says increasing a
position outside the allowlist reverts (the SDK models it as
`limitedBy: "voucherBlocked"` / a `VoucherMarketNotAllowed` placement revert).
On the live deployment the gate does not exist in practice:
`MarginBank.isVoucherMarketAllowed(pool)` returns `true` for **all 21**
markets, and a simulated position increase in a "blocked" market (ZEC) by an
account holding live voucher credit succeeds exactly like an allowlisted one
(BTC). Either the allowlist was never configured on this deployment or the
check is inert — in both cases the documented restriction is not in force.

## Steps to reproduce
Environment: Hideki testnet, chain 50383, `eth_call` reads/simulations at head.

1. Read the allowlist from the MarginBank for every perp pool:
   `MarginBank(0x478769E1177C0773c43A593DCCc4Ad15F15F1394).isVoucherMarketAllowed(pool)`
   for all 21 pools from the indexer market list.
2. Pick an account that actually holds voucher credit:
   `0xe102f8d60e4c5e17c9235aeecaf167594177b2be` —
   `VoucherDistributor.isClaimed` = true,
   `MarginBank.getCreditFloor(account)` = `1652152097651042046760` (≈1,652 USDso floor).
3. From that account, simulate a small resting limit bid that *increases*
   exposure in ZEC-PERP (docs: Paper USDso = **No**):
   `PerpPool(0x2a73129f26917124bd2a47a9b355c395c537f050).placeOrder(isBid=true,
   price=500×10^18, quantity=900000 [= min quantity], orderType=0, …)`.
4. Control: same shape on BTC-PERP (docs: Yes),
   `PerpPool(0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0)`, price 50,000×10^18,
   quantity 13000.

## Expected
Per docs/SDK: the ZEC increase reverts `VoucherMarketNotAllowed`, and
`isVoucherMarketAllowed(ZEC pool)` returns `false` (as for 1000PEPE, LIT,
PUMP, VVV, ZRO).

## Actual
- `isVoucherMarketAllowed` returns `true` for all 21 markets, including all
  six "No" markets.
- The ZEC placement simulation **succeeds**: `(success=true,
  orderId=313594649253063975184)`, indistinguishable from the BTC control
  `(success=true, orderId=7655398790589470386467)`.

## Evidence
- Full 21-market allowlist read: `evidence/api/voucher-allowlist.json`
  (all `true`).
- Placement simulations: `evidence/api/sim-orders2.json` (first run, result
  serialisation), `evidence/api/sim-boundary.json` entries
  `1_V_BTC_limit_ok` and `1b_V_ZEC_limit_ok` (re-run with explicit results).
- Account state: `evidence/api/sim-voucher.json` + worker log (claimed=true,
  credit floor 1,652.15 USDso).
- Docs text: docs.dreamdex.io → Trading → Perps on Testnet (market table,
  "Paper USDso" column) and Margin & Liquidations → Paper USDso ("Only some
  markets"); Developers → Perps on Testnet for Developers ("Paper USDso blocks
  some markets… `limitedBy: "voucherBlocked"`").

## Environment
- Network: Somnia Hideki testnet, chain ID 50383
- Contracts: MarginBank `0x478769E1177C0773c43A593DCCc4Ad15F15F1394`,
  ZEC pool `0x2a73129f26917124bd2a47a9b355c395c537f050`,
  BTC pool `0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0`
- SDK: `@somnia-chain/markets-sdk` 0.32.1
- Voucher account used: `0xe102f8d60e4c5e17c9235aeecaf167594177b2be`
