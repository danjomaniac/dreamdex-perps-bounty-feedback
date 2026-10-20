# Paper USDso is described three incompatible ways: a one-time SDK claim, a weekly Arena airdrop, and a voucher reclaimable after 14 days

- **Severity:** normal
- **Area:** docs (with app)

## Summary
The docs and the Perps Arena rules describe the same 1,000 paper USDso credit with three distribution models that cannot all be true at once: (1) a one-time, self-service claim any address can make through the SDK, (2) a weekly airdrop of 1,000 per wallet, and (3) a voucher the protocol can take back 14 days after it was granted. A builder implementing voucher-eligibility logic — or a trader planning around the credit — cannot tell from the documentation which model is real.

## Steps to reproduce
1. Developer guide, "Get test funds" (https://docs.dreamdex.io/developers/perps-testnet): "**Paper USDso.** Any address can claim the 1,000 USDso perps credit **once**, as long as it holds no perp positions or orders", followed by `getPerpVoucherEligibility` / `claimPerpVoucher` code. The trader guide (https://docs.dreamdex.io/trading/perps-testnet) likewise presents it as a single grant for joining Perps Arena.
2. Open the Perps Arena page the trader guide links to (https://app.testnet.dreamdex.io/competition) and read how paper funds are actually distributed: "Trade with a **weekly airdrop of 1,000 paper USDso per wallet**"; "Get paper funds — Get **weekly** paper USDso." The season-ROI worked example is built on "**four airdrops of 1,000 paper USDso**".
3. Read the Paper USDso section of "Margin & Liquidations" (https://docs.dreamdex.io/trading/perps-testnet/margin-and-liquidations): "A voucher can be taken back **14 days after it was granted**, but only while the account has no positions and no resting orders."

## Expected
One consistent description of how the 1,000 USDso credit is distributed (once vs weekly), and how any reclaim interacts with repeat airdrops.

## Actual
- If the credit is a one-time claim (developer guide), the Arena page's weekly airdrops and its four-airdrop worked example are wrong.
- If it is a weekly airdrop (Arena page), the developer guide's claim-once flow — and the `getPerpVoucherEligibility` logic a builder would write around it — is wrong, and the bot kit's own perps doc compounds it by calling the credit "Competition credit (the Perps Arena airdrop)".
- The 14-day reclaim (Margin & Liquidations) is never reconciled with a weekly cadence: nothing states whether each weekly voucher is reclaimed two weeks later, or whether reclaim applies only to the one-time claim.

## Evidence
- https://docs.dreamdex.io/developers/perps-testnet — "Any address can claim the 1,000 USDso perps credit once, as long as it holds no perp positions or orders"
- https://app.testnet.dreamdex.io/competition — "a weekly airdrop of 1,000 paper USDso per wallet"; "four airdrops of 1,000 paper USDso and final account equity of 5,200 give a season ROI of +30%"
- https://docs.dreamdex.io/trading/perps-testnet/margin-and-liquidations — "A voucher can be taken back 14 days after it was granted"
- https://github.com/somnia-chain/dreamdex-bot-kit/blob/main/docs/perps.md — "Competition credit (the Perps Arena airdrop) is a voucher"

## Environment
- Docs + app pages above, read 2026-10-20
- Network: Somnia Hideki testnet, chain ID 50383
- SDK: @somnia-chain/markets-sdk v0.32.1
