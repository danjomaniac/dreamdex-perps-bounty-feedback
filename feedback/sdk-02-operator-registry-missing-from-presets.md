# No shipped address preset contains `operatorPermissionsRegistry` — operator-grant reads always throw, perp stop orders can't build with the documented config

- **Severity:** major
- **Area:** SDK
- **Environment:** `@somnia-chain/markets-sdk` v0.32.1, Somnia Hideki testnet, chain 50383, indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, wallet (throwaway, build-only — nothing broadcast): `0x4c85a1cf2800948680567E6bEDF0C60655e3c39B`

## Summary

`client.isGloballyApproved(...)` and `client.isApprovedForPool(...)` take the OperatorPermissionsRegistry address **only** from `config.addresses.operatorPermissionsRegistry` (there is no per-call override). None of the three shipped presets — `SOMNIA_HIDEKI_TESTNET_ADDRESSES`, `SOMNIA_TESTNET_ADDRESSES`, `SOMNIA_MAINNET_ADDRESSES` — contains that field, so with the documented zero-setup config both reads **always** throw `NotConfiguredError`, on every network. The same missing field makes `trader.buildPlacePerpStopOrder(...)` / `placePerpStopOrder(...)` throw unless the caller hand-passes `operatorRegistry`.

The registry is deployed and discoverable: on Hideki it is `0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a` (dreamDEX docs, Contracts table), and the SDK itself can read it from any pool via `client.getOperatorPermissionsRegistry(pool)` — which returned exactly that address live. The presets simply never include it.

## Steps to reproduce

```js
import { SomniaMarkets, SOMNIA_HIDEKI_TESTNET_ADDRESSES, PLACE_ORDER_FOR_SELECTOR } from "@somnia-chain/markets-sdk";
import { hidekiTestnet } from "@somnia-chain/markets-sdk/chains";

const markets = new SomniaMarkets({
  chain: hidekiTestnet,
  indexerUrl: "https://hideki-dev.smk.somnia.host/v1/graphql",
  addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES, // documented preset
  privateKey: "0x…", // any key
});
const [btc] = await markets.client.listPerpMarkets({ baseSymbol: "BTC" });

await markets.client.getOperatorPermissionsRegistry(btc.poolAddress);
// -> "0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a"  (works — read from the pool)

await markets.client.isGloballyApproved({ owner, operator: btc.stopRegistry, selector: PLACE_ORDER_FOR_SELECTOR });
// -> throws NotConfiguredError

await markets.client.isApprovedForPool({ pool: btc.poolAddress, owner, operator: btc.stopRegistry, selector: PLACE_ORDER_FOR_SELECTOR });
// -> throws NotConfiguredError

await markets.trader.buildPlacePerpStopOrder({ registry: btc.stopRegistry, pool: btc.poolAddress, isBid: false, quantity: 0n, triggerPrice: 50000000000000000000000n, triggerOperator: 1, stopOrderType: 1 });
// -> throws NotConfiguredError
```

## Expected

With the shipped Hideki preset, operator-grant reads and perp stop-order placement work (the preset is documented as "pass as `config.addresses` for a zero-setup start"), or the reads resolve the registry from the pool the way `getOperatorPermissionsRegistry` already does.

## Actual

```
NotConfiguredError: @somnia-chain/markets-sdk: an operator grant read — needs addresses.operatorPermissionsRegistry
NotConfiguredError: @somnia-chain/markets-sdk: a perp stop order — needs operatorRegistry or addresses.operatorPermissionsRegistry
```

Control run: repeating the identical calls with `{ ...SOMNIA_HIDEKI_TESTNET_ADDRESSES, operatorPermissionsRegistry: "0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a" }` makes all three succeed — the reads return `false` and `buildPlacePerpStopOrder` returns a well-formed unsigned stop order (value `150000000000000000` = 0.15 STT, plus the `setOperatorApprovalGlobal` approval call). So the only defect is the missing preset field (verified for all three presets by inspecting the exported objects).

## Evidence

- Live run with preset-vs-control comparison: `evidence/sdk/test-live7-output.txt`
- Pool-resolved registry proof: `evidence/sdk/test-live4-output.txt` (`getOperatorPermissionsRegistry(BTC pool)` → `0x6B2FbfeD328FF5F20C0BE7D3C3437613aFFE8D3a`)
