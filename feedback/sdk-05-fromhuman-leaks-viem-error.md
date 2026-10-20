# `fromHuman` leaks viem's `InvalidDecimalNumberError` instead of an SDK error class, breaking documented `instanceof` error branching

- **Severity:** normal
- **Area:** SDK
- **Environment:** `@somnia-chain/markets-sdk` v0.32.1, any network (pure function, reproduced offline; no chain access needed)

## Summary

The SDK documents a closed error vocabulary: "every failure the SDK raises is one of these classes, so callers branch with `instanceof` instead of matching message strings" (`errors.ts`), and `fromHuman`'s own implementation guards non-finite numbers specifically so that "viem's `InvalidDecimalNumberError` — a third-party error crossing our boundary" cannot escape. But the guard only covers `NaN`/`Infinity`. Finite numbers in exponent notation and malformed strings still reach viem's `parseUnits` and throw the raw third-party error, which is **not** an instance of `SomniaMarketsError` (or `InvalidInputError`), so the documented branching pattern misses it:

- `fromHuman(1e21, 18)` → `Value.InvalidDecimalNumberError: Value \`1e+21\` is not a valid decimal number.` (Number→string gives `"1e+21"`, and `toFixed(18)` also returns `"1e+21"` for values ≥ 1e21, so the SDK's `humanToDecimalString` cannot save it.)
- `fromHuman("abc", 6)` and `fromHuman("", 6)` → same leaked error.

The SDK already solved this exact problem on the sibling path: `floorToRaw` (used for grid-floored quantities) explicitly throws `InvalidInputError` for exponent-notation magnitudes, with a comment noting the non-strict path "also rejects these (parseUnits refuses exponent notation)" — but the rejection it produces is the untyped viem error, not the SDK's typed one. `fromHuman` is also what the unified facade's price conversion (`Structs.toRaw`) delegates to, so the leak reaches `exchange.createOrder(...)` inputs as well.

## Steps to reproduce

```js
import { fromHuman, SomniaMarketsError } from "@somnia-chain/markets-sdk";

try { fromHuman(1e21, 18); } catch (e) {
  console.log(e.name);                          // "Value.InvalidDecimalNumberError"
  console.log(e instanceof SomniaMarketsError); // false  <- documented branching misses it
}
try { fromHuman("abc", 6); } catch (e) { console.log(e.name, e instanceof SomniaMarketsError); }
// -> "Value.InvalidDecimalNumberError" false

try { fromHuman(NaN, 6); } catch (e) { console.log(e.name, e instanceof SomniaMarketsError); }
// -> "InvalidInputError" true   (the guarded case — inconsistent with the two above)
```

## Expected

All invalid `fromHuman` inputs throw `InvalidInputError` (an `SomniaMarketsError` subclass), as the non-finite case already does and as `floorToRaw` does for the same magnitude class.

## Actual

Exponent-notation finite numbers (≥ 1e21, any `decimals`) and non-numeric strings throw viem's `Value.InvalidDecimalNumberError`, which callers catching SDK error classes — per the SDK's own documented pattern — will not catch.

## Evidence

- Reproduction output with `instanceof` checks: `evidence/sdk/fromhuman-output.txt`; broader conversion battery: `evidence/sdk/test-offline-output.txt`
