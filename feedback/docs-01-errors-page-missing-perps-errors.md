# The Errors page — linked as "the full list of order errors" — contains no perps errors at all

- **Severity:** major
- **Area:** docs

## Summary
The developer guide for perps sends builders to the Contracts → Errors page for "the full list of order errors", but that page documents only SpotPool and SpotStopOrderRegistry errors. Every perps-specific error a builder will actually hit on Hideki — including one the guide itself names without ever giving its selector — is missing, so a reverted perp transaction cannot be decoded using the documentation the guide points to.

## Steps to reproduce
1. Open "Perps on Testnet for Developers" (https://docs.dreamdex.io/developers/perps-testnet), section "An IOC that fills nothing reverts". The page ends with: "The full list of order errors is on the Errors page."
2. On the same page, note the perps errors the guide itself relies on:
   - "`placeOrderFor` reverts with `InsufficientCollateral()` (`0x3a23d825`)"
   - "reverts with `MaxPositionSizeExceeded()` (`0x714b36d6`)"
   - "Orders anywhere else revert with `IsolatedMarketBlocked`" (no selector given anywhere in the docs)
3. Follow the link to the Errors page (https://docs.dreamdex.io/developers/contracts/errors) and search the page for `InsufficientCollateral`, `MaxPositionSizeExceeded`, `IsolatedMarketBlocked`, or even the word "Perp".

## Expected
The Errors page lists the perps (PerpPool / MarginBank) errors with their selectors — at minimum the three errors the developer guide names, including a selector for `IsolatedMarketBlocked`.

## Actual
The Errors page has exactly three sections — "SpotPool errors", "SpotStopOrderRegistry errors", and "Preflight (HTTP API) reverts". A full-text search of the page returns zero occurrences of "Perp", "InsufficientCollateral", "MaxPositionSizeExceeded", or "IsolatedMarketBlocked" (verified 2026-10-20). The only perps error from the guide that appears is `ImmediateOrCancelNoFill()` (`0xd48c4403`), which is shared with spot. A builder whose perp order reverts with `IsolatedMarketBlocked` cannot find its selector, or any perps liquidation/margin/voucher errors, anywhere in the documentation.

## Evidence
- Developer guide: https://docs.dreamdex.io/developers/perps-testnet — quotes above.
- Errors page: https://docs.dreamdex.io/developers/contracts/errors — section headings "SpotPool errors" / "SpotStopOrderRegistry errors" / "Preflight (HTTP API) reverts"; zero matches for the four search terms above.
- The gap is structural: the whole Contracts reference (Addresses, Specifications, Functions, Events, Types, Errors) contains no perps content; the Addresses page lists only mainnet (5031) and Shannon testnet (50312), with no Hideki (50383) section.

## Environment
- Docs: https://docs.dreamdex.io/developers/perps-testnet and https://docs.dreamdex.io/developers/contracts/errors, read 2026-10-20
- Network: Somnia Hideki testnet, chain ID 50383
- SDK: @somnia-chain/markets-sdk v0.32.1 (the version the guide specifies)
