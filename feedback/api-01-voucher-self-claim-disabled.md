# Voucher self-claim always reverts `SelfClaimDisabled()` — documented claim flow is dead on Hideki

## Severity
major

## Area
contracts

## Summary
The developer guide and the SDK both say any address can claim the 1,000 USDso
perps credit once by calling the voucher claim (`trader.claimPerpVoucher` →
`VoucherDistributor.claim()`). On the live Hideki deployment, `claim()` reverts
with `SelfClaimDisabled()` (selector `0x3e31dcc2`) for **every** caller tested —
fresh wallets, active unclaimed traders, already-claimed accounts, even the
operator that runs the batch grants. The SDK's `getPerpVoucherEligibility` has no
model of this state and returns `eligible: true` for a fresh address, so the
documented onboarding flow (check eligibility → claim) succeeds at step 1 and
always fails at step 2. In practice the credit only reaches accounts through an
operator batch grant.

## Steps to reproduce
Environment: Hideki testnet, chain 50383, RPC
`https://api.hideki2.infra.testnet.somnia.network`. All calls are `eth_call`
simulations at chain head (identical execution path and revert data to a mined
transaction).

1. Read the campaign — it is live and funded:
   - `MarginBank(0x478769E1177C0773c43A593DCCc4Ad15F15F1394).getCreditGranter()`
     → `0x2ACD9EFdA721450906bfE1bef3edE269636E42C4` (the documented VoucherDistributor)
   - `VoucherDistributor.getConfig()` → grantAmount `1000000000000000000000`
     (1,000 USDso), campaignStart 1790553600, campaignEnd 1798477478 (window open)
   - `VoucherDistributor.budgetRemaining()` → `437221000000000000000000000`
2. Take a brand-new address, e.g. `0x7FA972b0fb89633997a5CD927Ba3Be2BA094791C`
   (zero balance, no positions, no orders, `isClaimed` = false). Per the docs
   ("Paper USDso: Any address can claim the 1,000 USDso perps credit once, as
   long as it holds no perp positions or orders") and the SDK's eligibility
   ordering, this address is eligible.
3. Simulate `VoucherDistributor.claim()` from that address (viem
   `simulateContract`, or the SDK's `trader.claimPerpVoucher`).

## Expected
Claim succeeds and credits 1,000 USDso to the caller's MarginBank balance
(the flow in "Perps on Testnet for Developers → Get test funds").

## Actual
Reverts with error selector `0x3e31dcc2` = `SelfClaimDisabled()`. The same
revert is returned for every sender tried:
- fresh wallet `0x7FA972b0fb89633997a5CD927Ba3Be2BA094791C`
- fresh wallet `0x7080Ce616bF17A610dADb65cA8c46238c95296b5`
- active trader, never claimed `0x8f0843bc18a8c95488839e71086ffd95f08cfde6`
- active traders, already claimed `0xe102f8d60e4c5e17c9235aeecaf167594177b2be`,
  `0x0f9abaae1d4dd8b5780d515498e186b928ed4389` (note: already-claimed accounts
  hit `SelfClaimDisabled` before any `AlreadyClaimed` check)
- the batch-grant operator `0x792182e6B2bd50dEB22dc5436F7C518b34F9bD12`
- arbitrary address `0x1234567890123456789012345678901234567890`

Meanwhile the only credits observed on-chain arrive via an operator batch
call: tx `0xdea4d4a2ef9792edef96f95c1e51ee3d3f780248e005fbc5fc0eef15da139613`
calls the distributor with selector `0x0677d0a3` and an array of 135 recipient
addresses, each credited 1,000 USDso (indexer `MarginEvent` kind `Credited`).

Additionally, the SDK's `getPerpVoucherEligibility` (v0.32.1, `src/perp/voucher.ts`)
checks dormant / notStarted / ended / notFlat / mainFunded / alreadyClaimed /
budgetExhausted — in that order, mirroring what it believes `claim()` checks —
and has no representation of the self-claim-disabled state, so it affirmatively
reports `eligible: true` for addresses whose claim will always revert.

## Evidence
- Simulation results: `evidence/api/sim-voucher.json` (config, budget, claim
  simulation), follow-up runs sim2/sim4 in the worker log
  (`workers/api-LOG.md`): identical `0x3e31dcc2` revert for 7 distinct senders.
- Selector identification: `cast sig "SelfClaimDisabled()"` → `0x3e31dcc2`.
- Batch-grant transaction: `0xdea4d4a2ef9792edef96f95c1e51ee3d3f780248e005fbc5fc0eef15da139613`
  (to VoucherDistributor, status 1).
- Docs text: docs.dreamdex.io → Developers → Perps on Testnet for Developers,
  "Get test funds" section.

## Environment
- Network: Somnia Hideki testnet, chain ID 50383
- Contracts: VoucherDistributor `0x2ACD9EFdA721450906bfE1bef3edE269636E42C4`,
  MarginBank `0x478769E1177C0773c43A593DCCc4Ad15F15F1394`
- SDK: `@somnia-chain/markets-sdk` 0.32.1 (source read for the eligibility logic)
- Test wallet: `0x7FA972b0fb89633997a5CD927Ba3Be2BA094791C`
