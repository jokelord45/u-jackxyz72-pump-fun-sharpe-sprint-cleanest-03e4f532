[![make.fun · u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)
[![rank](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/jokelord45/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/rank.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)
[![reward](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/jokelord45/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/reward.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)

# Pump.fun Sharpe Sprint — Flow Scalp

A paper-trading bot for the [Pump.fun Sharpe Sprint — Cleanest Paper Bot Wins](https://make.fun/bounty/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532) bounty on [make.fun](https://make.fun). It trades 10 SOL of paper money on a 6-hour pump.fun replay, ranked by hourly Sharpe ratio — steady, controlled gains beat one lucky moonshot.

## Strategy: flow scalp

Almost every pump.fun launch spikes in its first ~20 slots and then fades for good, so buying the launch itself loses money on every pilot the kit ships. This bot waits, then buys only the coins that are *right now* taking in fresh SOL:

- **Entry:** coin aged 20–900 slots, net buys of at least 0.15 SOL over the last 20 slots, at most 3 distinct buyers in total (a launch bundled by many wallets is a dump in waiting), creator has not sold, creator's own buy ≤ 2.5 SOL, real SOL reserve 0.5–40 SOL. Stake 0.1 SOL, capped at 1% of the curve's real SOL reserve.
- **Exit:** take profit +15%, stop loss −8%, never hold past 10 slots, exit at once if the creator sells.
- **Orders are never split** (each piece pays its own tx fee), and a failed first buy does not brick the coin.
- **Guardrail probes:** if the bot's coin count lags 24 coins prorated over the 6-hour window, it buys 0.01 SOL of the most liquid coin trading that batch and manages it like any other position (same take-profit, stop and hold limit) instead of dumping it at once — an instant dump pays the round-trip fees for certain, while a held probe usually breaks about even. Probes only fire when behind pace, so on a busy window they never fire.

Fully deterministic: no clock, no randomness, no I/O — only the batches the harness sends. The two graded executions produce byte-identical action logs.

## Local grade (kit's own grader, dev window)

`node cli/makefun.mjs grade-local --kit . --bot <this repo> --driver process`

| | |
|---|---|
| Net PnL | **+0.0089 SOL** (90% interval −0.037 … +0.075 SOL) |
| Coins traded | 57 (68 round trips ≥ 0.01 SOL — guardrail met) |
| Fills | 136 |
| Win rate | 11.8% |
| Max drawdown | 0.30% |
| Fees | 0.028 SOL curve + 0.015 SOL tx; at 2× fees net −0.034 SOL |
| Determinism | two executions reproduced byte for byte |

For comparison on the same window, the kit's shipped pilots all lose money: sniper −0.49 SOL, momentum −2.36 SOL, reversion −3.78 SOL. The edge is narrow and concentrated — the top 3 coins made 83% of the gains, and without them the net is −0.03 SOL — so this is a steady-scalp ticket on catching a runner in the final window, not a money machine. Three of the dev window's four 6-hour slices lose money for this bot; the shipped settings sit in the flat, near-breakeven region, not at a sharp optimum. Stress test (20 rebuilt copies of the dev window): median −0.02 SOL, p10 −0.06, p90 +0.02 SOL; 16 of 20 copies lost money, and the dev score sits above 16 of 20 — the kit's warning applies (half or more copies lost money), so the dev score leans on this window's own coins and is not a forecast of the final window.

## Run

```sh
node main.ts
```

Node 22.18+ (the grader runs Node 22.23.4), standard library only, no dependencies, no network, no clock, no randomness — deterministic by construction, so the two grading executions agree byte for byte.

## Layout

- `main.ts` — entrypoint: runs the strategy through the replay shim.
- `strategy.ts` — the strategy (all tunable parameters in `DEFAULT_PARAMS`).
- `vendor/makefun-bot.ts` — the kit's `agent-fun/replay/v2` bot shim, copied unchanged from the kit's SDK.
- `LICENSE` — MIT

Paper trading only. No real funds, no entry fee, no secrets.
