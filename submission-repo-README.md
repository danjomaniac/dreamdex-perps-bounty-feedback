# dreamDEX Perps Builder Bounty — Feedback Submission

Researcher: @danjomaniac
Track: Feedback (API / SDK / docs / contracts)
Network: Somnia Tokyo testnet (chain 50383, "Hideki"), `@somnia-chain/markets-sdk` v0.32.1

16 validated, reproducible items — each in its own file under `feedback/`, in the
program's format (title, severity, area, steps to reproduce, expected/actual,
evidence, environment).

## Major (6)

| File | Item |
|---|---|
| `feedback/api-01-voucher-self-claim-disabled.md` | `VoucherDistributor.claim()` reverts `SelfClaimDisabled()` for every caller class while the SDK still reports callers as eligible — documented onboarding flow is dead |
| `feedback/api-02-voucher-allowlist-not-enforced.md` | Voucher market allowlist not enforced on-chain: all 21 markets report allowed, including 6 the docs mark blocked |
| `feedback/api-03-indexer-markprice-stale.md` | Indexer `Market.markPrice` stale 19–44 minutes across markets (deviations up to 138 bps) while trades execute seconds earlier |
| `feedback/sdk-01-perp-symbol-casing-docs.md` | Shipped SDK docs use `BTC/USDSO:USDSO`; live symbols are `BTC/USDso:USDso` — every documented unified perp call throws "unknown symbol" |
| `feedback/sdk-02-operator-registry-missing-from-presets.md` | No SDK address preset contains `operatorPermissionsRegistry` — approval/stop-order calls always throw `NotConfiguredError` |
| `feedback/docs-01-errors-page-missing-perps-errors.md` | Docs Errors page omits all perps errors although the dev guide points builders there |

## Normal (10)

`api-04` indexer `Order.pool` null on all perp orders · `api-05` batch
`amendOrders` not all-or-nothing despite docs · `api-06` docs OI/cap/builder-fee
numbers stale vs on-chain · `api-07` `placeOrderFor`/`cancelOrderFor` unusable
by EOAs (`OnlyApprovedContracts`) · `sdk-03` PERPS.md stale market list + broken
WBTC example · `sdk-04` SDK rejection-reason names contradict its own error ABI
· `sdk-05` `fromHuman(1e21, 18)` leaks viem's raw error class · `docs-02`
dreamBot Builder page covers Spot + Event Contracts only · `docs-03` Paper USDso
described three incompatible ways · `docs-04` Arena eligibility requirements
missing from onboarding docs.

All findings were reproduced live against the testnet API/SDK/contracts on
2026-10-20. No security vulnerabilities are included in this repository, per the
program's rules.
