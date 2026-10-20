# `PERP_ORDER_REJECTION_REASON` names disagree with the contract error names used everywhere else in the SDK and docs

- **Severity:** normal
- **Area:** SDK
- **Environment:** `@somnia-chain/markets-sdk` v0.32.1, Somnia Hideki testnet, chain 50383 (name table is chain-independent; live rejection rows verified against the Hideki indexer)

## Summary

`client.listPerpOrderRejections()` maps the on-chain `OrderRejectionReason` index to a name via the exported `PERP_ORDER_REJECTION_REASON` table. Two of its names do not match the names of the corresponding contract errors — the names used by the SDK's own `contractErrorsAbi`, by the SDK's other doc comments, and by the dreamDEX docs Errors page:

| Index | `PERP_ORDER_REJECTION_REASON` (what `listPerpOrderRejections` returns) | Contract error name (SDK `contractErrorsAbi` + docs Errors page) |
|---|---|---|
| 1 | `AlreadyExpired` | `OrderAlreadyExpired()` — selector `0x3154078e` |
| 4 | `FillOrKillUnfillable` | `FillOrKillNotFillable()` — selector `0xc04ad919` |

A bot that branches on the documented error names — e.g. `if (r.reason === "FillOrKillNotFillable")` — silently never matches, because the same SDK spells it `FillOrKillUnfillable` on this one surface. (Indices 0, 2, 3, 5 — `None`, `SelfMatchCancelTaker`, `PostOnlyWouldCross`, `ImmediateOrCancelNoFill` — do match the error names.)

## Steps to reproduce

```js
import { PERP_ORDER_REJECTION_REASON, contractErrorsAbi } from "@somnia-chain/markets-sdk";

console.log(PERP_ORDER_REJECTION_REASON);
// -> ["None","AlreadyExpired","SelfMatchCancelTaker","PostOnlyWouldCross","FillOrKillUnfillable","ImmediateOrCancelNoFill"]

console.log(contractErrorsAbi.filter(e => e.type === "error").map(e => e.name)
  .filter(n => n.includes("FillOrKill") || n.includes("Expired")));
// -> ["FillOrKillNotFillable", "OrderAlreadyExpired", …]  (no "FillOrKillUnfillable", no "AlreadyExpired")
```

Live check that the table is the one served to callers: `client.listPerpOrderRejections({ limit: 50 })` on Hideki returns rows with `reasonRaw: 3, reason: "PostOnlyWouldCross"` — the mapping is live; indices 1 and 4 will surface the divergent spellings as soon as those rejections occur.

## Expected

One spelling per reason across the SDK: the rejection names match the contract error names (`OrderAlreadyExpired`, `FillOrKillNotFillable`), as the dreamDEX Perps developer docs promise ("its `OrderRejected` log carries reason `5`, which `client.listPerpOrderRejections` returns by name" — the name should be the contract's).

## Actual

Two spellings for the same reasons inside one package: `FillOrKillUnfillable` / `AlreadyExpired` in `perp/history.js` (the rejection table) vs `FillOrKillNotFillable` / `OrderAlreadyExpired` in `contractErrorsAbi.js`, `tradeAbi.js` comments, `trade.d.ts`, and the docs Errors page (selectors `0xc04ad919` / `0x3154078e` verified by computing `toFunctionSelector` over the ABI signatures).

## Evidence

- Offline verification output (ABI membership + selector computation; only `FillOrKillNotFillable()` = `0xc04ad919` and `OrderAlreadyExpired()` = `0x3154078e` match the docs Errors page): `evidence/sdk/rejection-names-output.txt`, `evidence/sdk/test-offline-output.txt`
- Live rejection rows using the same table: `evidence/sdk/rejections-output.txt`
