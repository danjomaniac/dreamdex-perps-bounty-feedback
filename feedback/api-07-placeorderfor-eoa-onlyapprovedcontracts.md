# `placeOrderFor` / `cancelOrderFor` revert `OnlyApprovedContracts` for every EOA — the documented bot-key raw flow cannot run

## Severity
normal

## Area
API

## Summary
The developer guide presents `placeOrderFor` as the raw route by which a
linked bot key trades a user's wallet ("a bot key placing orders for a linked
wallet with `placeOrderFor`", "the key can then place, cancel and reduce
orders for that wallet, and nothing else"). On the deployed PerpPools, every
EOA call to `placeOrderFor` — and `cancelOrderFor` — reverts with
`OnlyApprovedContracts` (selector `0x3fb0ba2e`), regardless of the owner
argument: owner = a third party, owner = a voucher-funded stranger, and even
owner = the caller itself. So the *For* entry points are, in this deployment,
usable only by approved contracts; an EOA bot key following the guide hits a
wall with an error that the guide never mentions. (Owner-direct `placeOrder` /
`cancelOrder` work as expected, and ownership checks on them are enforced —
strangers get `IncorrectSender`.)

## Steps to reproduce
Environment: Hideki testnet, chain 50383, `eth_call` at head, BTC pool
`0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0`, caller
`0x7080Ce616bF17A610dADb65cA8c46238c95296b5` (an ordinary EOA).

1. `placeOrderFor(owner=caller, isBid=true, price=50000×10^18, quantity=13000,
   expire=+1h, orderType=0, …)` → revert `0x3fb0ba2e`.
   (The RPC node decodes this selector as `OnlyApprovedContracts`.)
2. Same call with `owner=0x7FA972b0fb89633997a5CD927Ba3Be2BA094791C` (another
   wallet) and with `owner=0xe102f8d60e4c5e17c9235aeecaf167594177b2be` (a
   funded account) → identical revert.
3. `cancelOrderFor(owner, orderId)` for a real resting order → identical
   revert.
4. Controls: the same parameters via `placeOrder` from the funded owner
   succeed; `cancelOrder` by the owner succeeds; `cancelOrder` by a stranger
   reverts `IncorrectSender(caller, owner)` (`0xf5e39c1f`).

## Expected
Per the guide, a bot key (an EOA) linked via "Link a Bot" /
OperatorPermissionsRegistry can place and cancel for the linked wallet through
the *For* functions, and the guide's "Raw and operator calls" section details
exactly their pitfalls (quantity scale, gas, IOC semantics) — implying they
are callable by such a key.

## Actual
The *For* functions are gated to approved contracts only on this deployment;
no EOA can call them at all, and neither the gate nor the intermediary
contract a bot is supposed to use instead is documented.

## Evidence
- Simulations: `evidence/api/sim-orders-auth.json`, `evidence/api/sim-orders2.json`,
  `evidence/api/sim-cancel-amend.json` (cancelOrderFor).
- Raw revert capture (cast): `Error: execution reverted: OnlyApprovedContracts …
  data: "0x3fb0ba2e"`.
- Docs text: docs.dreamdex.io → Developers → Perps on Testnet for Developers
  ("Trade an app wallet with a bot", "Raw and operator calls").

## Environment
- Network: Somnia Hideki testnet, chain ID 50383
- Contracts: BTC PerpPool `0x4181dd08e73bbcdd1ba3e615a18f6abd941b94c0`,
  OperatorPermissionsRegistry `0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a`
- Test wallet: `0x7080Ce616bF17A610dADb65cA8c46238c95296b5`
