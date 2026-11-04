# Building a trading bot for the SIG Predictions Cup

[Interactive report](https://tristandarnell.github.io/SIG-Predictions-Cup-Writeup/) · [Technical notes](docs/technical-retrospective.md) · [Data and methods](docs/methods.md)

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="figures/01-equity-and-rank-dark.png">
  <img src="figures/01-equity-and-rank.png" alt="Account value and leaderboard rank during the competition">
</picture>

I ran a Rust trading bot in the SIG Predictions Cup from October 1 to November 4, 2026. The account started with **100,000 SUSQies** and ended trading in **sixth place**, with **762,110 in marked account value (+662.1%)**. It peaked at 1.055 million and reached second place during the month.

These are virtual competition funds. Positions are still awaiting settlement, so the final value and ranking can change.

## How it traded

The bot compared SIG contracts with the same outcomes on **Kalshi and Polymarket US**. It took prices that looked cheap or expensive against those references and left orders deeper in the book to catch large trades. After a fill, it tried to close the position when the price gap narrowed.

That worked best when the move was temporary. Some positions stayed open much longer than expected, tying up cash that could have funded other trades.

The engine used **Rust and Tokio**, with WebSocket updates and REST reads to recover missed fills and check account state. Python handled research and monitoring. Manual trades also changed the account, which became important later.

```mermaid
flowchart LR
    F[Kalshi + Polymarket US] --> V[Contract and quote checks]
    V --> E[Rust trading engine]
    E --> X[SIG exchange]
    X --> R[Orders, fills and positions]
    R --> E
    R --> J[Logs and Python research]
```

Small data errors could produce convincing trade signals. One mapping linked a governor contract to a lieutenant-governor market. A thin outside order book could also make its midpoint misleading. The bot needed checks on the contract, quote age, spread, and disagreement between sources before using a price.

## Getting orders and cash right

The account was already marked at about **806,000 by the end of October 2**. Keeping it trading was harder than the early growth suggested.

Some fills arrived late through REST. Others appeared in both order responses and WebSocket messages. Counting one twice could make the bot send the wrong exit. An order request could also time out while the order remained live at the exchange.

The fixes included duplicate handling, retries that reused the same request key, and reservations for pending orders. Cancels received priority over new orders.

Cash was another constraint. In Rhode Island Senate, the bot traded an extra NO position around a protected core holding. It sold YES into price spikes, then tried to buy it back after the price fell. Even a buyback that reduced exposure still needed available cash under the venue's rules.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="figures/03-cash-and-basis-dark.png">
  <img src="figures/03-cash-and-basis.png" alt="Cash and the cost of remaining holdings over the final four days">
</picture>

Before election night, the regular sweep orders were tapered and cash recovered to roughly **500,000**. Almost all of it was then spent on new positions.

## A market-making idea that failed testing

Tighter quotes looked profitable in an initial replay: about **+1,250 per day** with an assumed place at the front of the queue. With a more realistic queue model, that became **−414 per day**. A later shadow run lost about **213 over 39 hours**.

Those were simulated results. The dedicated experiment stayed in shadow mode. Other traders' orders could absorb the ordinary flow ahead of ours, leaving us with fills from trades that kept moving through the level. Replacing orders too often also lost queue priority.

## Election night

The account entered the evening near **1.016 million**, in fourth place. Two things went wrong.

In MN-01, a manual purchase changed a small short into a large long, but an old exit setting remained at **0.01**. The bot then sold **50,000** newly purchased YES contracts under that stale rule, losing **25,939.53 against their purchase cost**, before any unshown charges. The exit setting should have reset when the position changed direction.

The larger decline came after trading stopped. At the final filled-order-count update, just after **00:47 EST**, account value was **1,046,386.23**. It then fell by **284,276.56** through the first post-close snapshot, without further recorded trades.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="figures/02-election-night-dark.png">
  <img src="figures/02-election-night.png" alt="Account value falling after the final recorded trades on election night">
</picture>

Cash was down to **0.39**. Some apparently separate positions were also the same bet: Democratic YES and Republican NO both depended on a Democratic win. The next version needs limits across each race, plus an exit reset whenever a manual trade reverses a position.

## Code and data

Claude and Codex helped with coding, research, and debugging. I chose allocations, approved deployments, changed limits, and placed manual trades.

The account recorded **27.04 million in volume across 18,027 orders with fills**. The charts use **24,064 leaderboard snapshots**. The [technical notes](docs/technical-retrospective.md) cover the system and incidents in more detail; the [methods](docs/methods.md) explain the calculations and gaps in the records.

<details>
<summary>Watch the account replay</summary>

![Replay of recorded account value and rank](figures/05-equity-replay.gif)

The recorder missed the first 11.45 hours, so the initial 100,000 is shown separately.

</details>

*Write-up format inspired by [IHateVolatility's SIG Prediction Cup report](https://github.com/IHateVolatility/SIG-Prediction-Cup).*
