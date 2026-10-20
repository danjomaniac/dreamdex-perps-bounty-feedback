# `amendOrders` is not all-or-nothing: rejected replacements return id 0 and the batch commits

## Severity
normal

## Area
contracts

## Summary
The developer guide states: "An all-or-nothing `amendOrders` reverts with
`AmendReplacementRejected(requestIndex, reason)`." The deployed PerpPool does
the opposite: a batch amend containing a replacement that cannot be placed
(e.g. an IOC that finds no fill) does **not** revert — it returns order id `0`
for the failed entry and commits the rest of the batch. The singular
`amendOrder` with the same failing replacement does revert, but with an error
selector (`0xb16f4abb`) that matches neither the documented
`AmendReplacementRejected(uint256, uint8)` (selector `0x4723981c`, per the SDK's
own contract-error table) nor any error in that table.

## Steps to reproduce
Environment: Hideki testnet, chain 50383, `eth_call` simulations at head,
BTC pool `0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0`, caller = owner of two
resting orders (`0x47537e3a3ea6e9391f3b26ea45618db6828c8bff`, order ids
`11104939932373156538823` and `7968993439842532764101`).

1. Batch `amendOrders` with two requests: [0] valid resting limit replacement,
   [1] IOC replacement priced away from the book (nothing to fill).
   Result with `alwaysPlace=true`: **success**, returns
   `["9389392733518168238574", "0"]` — first entry replaced with a new id,
   second entry silently 0, no revert. Same with `alwaysPlace=false`: success,
   returns `[0, 0]`.
2. Batch with a nonexistent `oldOrderId` (424242424242424242): success,
   returns `[0]`.
3. Singular `amendOrder` with the IOC replacement: reverts with raw data
   `0xb16f4abb0000…025a 0000…62a9c7` — neither the documented
   `AmendReplacementRejected` selector (0x4723981c) nor argument shape
   (requestIndex, reason).

## Expected
Per the developer guide ("Raw and operator calls" section): a rejected
replacement in `amendOrders` reverts the whole batch with
`AmendReplacementRejected(requestIndex, reason)`.

## Actual
`amendOrders` behaves like `placeOrders`: per-entry soft failure (id 0), batch
commits. Integrators relying on the documented all-or-nothing semantics will
believe an amend succeeded atomically when the replacement was in fact dropped.
Authorization itself is fine (a stranger's amend reverts `IncorrectSender`,
`0xf5e39c1f`).

## Evidence
- Simulations: `evidence/api/sim-cancel-amend.json` (mixed batch, alwaysPlace
  true), `evidence/api/sim-amend2.json` (alwaysPlace false variants, singular
  amend, bogus id).
- Raw revert capture of the singular amend (cast): data `0xb16f4abb…` shown in
  `workers/api-LOG.md`.
- Docs text: docs.dreamdex.io → Developers → Perps on Testnet for Developers:
  "An all-or-nothing `amendOrders` reverts with `AmendReplacementRejected(requestIndex, reason)` instead."

## Environment
- Network: Somnia Hideki testnet, chain ID 50383
- Contract: BTC PerpPool `0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0`
- SDK: `@somnia-chain/markets-sdk` 0.32.1 (error table used for the selector
  comparison)
