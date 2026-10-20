# Perps getting-started docs omit Perps Arena's actual eligibility requirements (registration + holding 10 SOMI on mainnet)

- **Severity:** normal
- **Area:** docs (with app)

## Summary
The trader guide's onboarding flow says joining Perps Arena is how you receive the 1,000 paper USDso, and neither it nor any other perps doc mentions the competition's mandatory eligibility conditions. The Arena's own rules require registration **and holding at least 10 SOMI on Somnia mainnet continuously from registration until rewards are paid** — a user who follows the docs exactly can trade for four weeks and be ineligible for every reward without ever seeing the requirement.

## Steps to reproduce
1. Read the getting-started flow in "Perps on Testnet" (https://docs.dreamdex.io/trading/perps-testnet), step 2: "**Get paper USDso.** Join, the paper-trading competition, to receive 1,000 paper USDso to trade with." No eligibility conditions are stated in the trader guide, the developer guide, or the Margin & Liquidations page. The docs banner and the testnet app banner advertise the competition as "No deposit required."
2. Open the linked competition page (https://app.testnet.dreamdex.io/competition), "Are you eligible?" checklist: item 1 "Registered for Perps Arena", item 2 "Hold 10 SOMI on Somnia mainnet — Hold 10+ SOMI on Somnia mainnet in your dreamDEX mainnet wallet or in your sign-up wallet. Keep it there from registration until rewards are paid."
3. Read "Reward eligibility and fair play" on the same page: "To receive any weekly leaderboard, season leaderboard or raffle reward, you must: … Hold at least 10 SOMI on Somnia mainnet in your dreamDEX mainnet wallet or your sign-up wallet, continuously from registration until rewards are paid".

## Expected
The perps docs' onboarding (the pages a new user actually follows) state the registration step and the 10 SOMI mainnet holding requirement, or link to them where the paper-USDso step is described.

## Actual
The requirement appears only inside the competition page's rules. The docs present Arena participation as: sign in → join → receive paper USDso → trade. Nothing in the docs mentions SOMI, a mainnet wallet, or a continuous-holding condition, and the "No deposit required" framing in the docs/app banners sits directly against a mandatory mainnet token hold.

A related trap on the same rules page: the fair-play list makes "**Unauthorized paper funds** — Depositing paper USDso from outside your own weekly airdrop, or withdrawing paper USDso" a disqualification ground, while the developer guide instructs any address to obtain the credit directly via the SDK (`claimPerpVoucher`). The docs never explain how the documented SDK claim relates to the "weekly airdrop" for fair-play purposes.

## Evidence
- https://docs.dreamdex.io/trading/perps-testnet — step 2 quote above; no occurrence of a SOMI holding requirement anywhere in the perps docs.
- https://app.testnet.dreamdex.io/competition — eligibility checklist and "Reward eligibility and fair play" quotes above; fair-play "Unauthorized paper funds" quote above.
- https://docs.dreamdex.io/developers/perps-testnet — SDK claim flow (`markets.trader.claimPerpVoucher({ marginBank})`) presented for "any address", with no mention of Arena registration or fair-play limits.

## Environment
- Docs + app pages above, read 2026-10-20
- Network: Somnia Hideki testnet, chain ID 50383 (competition eligibility involves Somnia mainnet, chain ID 5031)
