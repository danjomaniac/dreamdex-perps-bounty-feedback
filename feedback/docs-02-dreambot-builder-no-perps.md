# Developer guide says the dreamBot Builder "sets up a perps bot for you" — the Builder only covers Spot and Event Contracts

- **Severity:** normal
- **Area:** docs (with app)

## Summary
The perps developer guide offers two ways to have a bot trade an app wallet: the open-source bot kit, and the dreamBot Builder, which it says "sets up a perps bot for you". The dreamBot Builder's own page scopes it to Spot and Event Contracts only — there is no perps flow there, so the second documented path is a dead end for perps builders.

## Steps to reproduce
1. Open "Perps on Testnet for Developers" (https://docs.dreamdex.io/developers/perps-testnet), section "Trade an app wallet with a bot": "The open-source dreamDEX bot kit does this when `OWNER_ADDRESS` is set to the linked wallet, and the dreamBot Builder sets up a perps bot for you."
2. Follow the page's "dreamBot Builder" link to https://app.testnet.dreamdex.io/dreambot-builder.
3. Read the page's own description of what it does.

## Expected
The dreamBot Builder offers a perps bot setup, as the developer guide claims — or the guide does not present it as a perps option.

## Actual
The Builder page states: "Build, test, and deploy dreamBots for **Spot and Event Contracts**, in five steps." Perps are not mentioned anywhere on the page. This is consistent with the bot kit repository (github.com/somnia-chain/dreamdex-bot-kit), where perps are served by the code-based `perp-*` strategies and the Builder is never presented as a perps path — only the developer guide makes that claim.

## Evidence
- Developer guide: https://docs.dreamdex.io/developers/perps-testnet — "and the sets up a perps bot for you."
- Builder page: https://app.testnet.dreamdex.io/dreambot-builder — "Build, test, and deploy dreamBots for Spot and Event Contracts, in five steps. Your private key never touches this widget."
- Bot kit README: https://github.com/somnia-chain/dreamdex-bot-kit — perps coverage is via `strategies/perp-*` (code), linked to `docs/perps.md`; no Builder-based perps setup.

## Environment
- Docs: https://docs.dreamdex.io/developers/perps-testnet, read 2026-10-20
- App: https://app.testnet.dreamdex.io/dreambot-builder, read 2026-10-20
- Network: Somnia Hideki testnet, chain ID 50383
