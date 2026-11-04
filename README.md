# Trading the midterms with Rust

[Interactive report](https://tristandarnell.github.io/SIG-Predictions-Cup-Writeup/) · [Technical retrospective](docs/technical-retrospective.md) · [Data and methods](docs/methods.md)

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="figures/01-equity-and-rank-dark.png">
  <img src="figures/01-equity-and-rank.png" alt="Observed account value and leaderboard rank across the SIG Predictions Cup">
</picture>

From October 1 to November 4, 2026, we built and operated a trading system for the SIG Predictions Cup. Starting with **100,000 SUSQies**, the account reached **762,109.67 in marked value** on the first post-close leaderboard: a **+662.11% marked return** and **provisional sixth place**. Its highest recorded value was 1.055 million, and its best observed rank was second.

The system combined a Rust execution engine, outside-market prices, passive orders, inventory management, and Python research tools. The venue recorded **27.04 million in volume across 18,027 orders with fills**. Along the way, the project became as much an exercise in reliable systems and capital allocation as in finding attractive prices.

These are virtual competition results, measured before settlement. Open positions remained at the close; the marked return is not realized cash profit. The [settlement rules](https://sig.thesuper.market/docs/settlement-and-payouts) determine the final standings.

<details>
<summary>Watch the account replay</summary>

![Animated replay of recorded account value, marked return, and rank](figures/05-equity-replay.gif)

The opening allocation is shown separately: the leaderboard recorder missed the first 11.45 hours. The full observed drawdown remains in the replay.

</details>

## From a price signal to an order

The initial idea was to compare SIG prices with matching contracts on **Kalshi and Polymarket US**. When a competition price offered enough room relative to a usable outside reference, the engine could take it. It could also rest orders deeper in the book, waiting for a large incoming trade to reach them.

That required more than matching market names. Contracts needed the same outcome and settlement terms; inverted outcomes needed consistent handling. The reference layer filtered stale, invalid, and overly wide quotes, checked disagreement, and retained a conservative price band. We corrected mapping mistakes during the competition, including governor contracts linked to lieutenant-governor references.

```mermaid
flowchart LR
    F[Kalshi + Polymarket US] --> V[Contract mapping and price checks]
    V --> E[Rust strategy and inventory engine]
    E --> O[Cash and order controls]
    O --> X[SIG exchange]
    X --> R[Fill and position reconciliation]
    R --> E
    R --> J[Journal and Python research]
```

The engine used **Rust and Tokio**, with WebSocket updates and REST recovery. Python supported research, monitoring, and operator tools. Human instructions and manual trades also affected the account, so reconciliation had to describe the whole account rather than only the engine's own intentions.

The passive strategy was built around a cycle: catch a temporary dislocation, hold the inventory, then recycle it when the gap narrowed. A large apparent entry advantage could still become a long holding period. If the price move contained new information, the expected rebound might never arrive.

## Cash became part of the strategy

By the last recorded observation on October 2, the account was marked at approximately 805,650. By October 6, it was 907,417. Growth was strong, but every new holding competed with the next trade for cash.

Rhode Island Senate inventory became a useful example. We held Republican NO exposure and developed a slice around a protected core: sell additional YES into a price spike, increasing NO exposure, then buy that added slice back if the price receded. Closing the slice still required available cash under the venue's rules. A position could be economically attractive and operationally difficult to recycle.

We added pending-order reservations, reconciliation, idempotent submission behavior, and cancellation priority. Delayed fills and unknown submission outcomes mattered directly to the strategy: an order could exist at the exchange before the local state reflected it, while a duplicate notification could make an inventory limit wrong in the opposite direction.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="figures/03-cash-and-basis-dark.png">
  <img src="figures/03-cash-and-basis.png" alt="Cash and remaining acquisition basis ahead of the trading close">
</picture>

Ahead of election night, the ordinary sweep ladders were tapered and cash recovered to roughly **500,000**. That reserve was subsequently committed almost entirely to inventory. The question had changed from whether we could find another price discrepancy to whether it was the best use of the remaining capital.

## The tests that changed the plan

Tighter market making was an appealing next step: trade more often, turn inventory faster, and collect smaller spreads. The queue model changed that conclusion.

An October 12 replay produced approximately **+1,250 per day** when it assumed we were first in the queue. With a more realistic queue model, the result was approximately **−414 per day**. A later forward shadow run was approximately **−213 over 39 hours**. These were research results, not live account returns; the dedicated experiment stayed shadow-only.

An attractive quote did not guarantee an attractive fill. Existing orders could absorb ordinary flow before ours traded. The flow that continued through the level could carry more information. Replacing an order also sacrificed its place in the queue.

We tested depth, refill timing, inventory limits, and reservations with those constraints in mind. The useful research outcome was sometimes a decision to leave an idea disabled. It was better to identify a favorable simulation's assumptions than to mistake its output for deployable profit.

## Election night

At 17:58 EST on November 3, the account was marked at approximately **1.016 million**, in fourth place. Two different problems shaped the final night.

First, a manual MN-01 purchase reversed a small short into a large long while an old exit pin remained active. The bot then sold 50,000 newly purchased YES contracts using that stale rule. Authoritative order responses verify a **25,939.53 gross matched-basis loss**, before any unshown charges. An exit instruction needed to belong to a particular position direction and lifecycle; the old numerical price survived a transition it should not have survived.

Second, the larger later decline occurred after the final recorded execution. At the last filled-order-count update, just after 00:47 EST on November 4, the account was marked at **1,046,386.23**. Volume and filled-order count then stayed fixed, and position snapshots showed unchanged quantities and entry averages from 00:51 onward.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="figures/02-election-night-dark.png">
  <img src="figures/02-election-night.png" alt="Election-night account value and the interval after the final recorded trading activity">
</picture>

The subsequent decline to the first post-close mark was **284,276.56**. It reflected repricing of carried inventory, rather than more executions during that interval. Cash had fallen to 0.39, so further cash-funded entries required an exit or payout first.

Several positions were different contracts expressing the same race outcome. Democratic YES and Republican NO were aligned exposure, not a hedge. The incident analysis therefore separated an execution-control failure from a portfolio-risk failure. Repairing one would not automatically repair the other.

## What the next version would improve

The next iteration would make three changes central to the design.

**One account, one position lifecycle.** Manual reversals and external fills should invalidate dependent exits until the account is reconciled. Order intent, ownership, direction, and reservations need to move together.

**Risk budgets tied to outcomes.** Limits should aggregate aligned YES and NO contracts within a race, then consider common election scenarios across races. Capital allocation should include the likely holding period and the cost of an exit.

**Valuations with visible coverage.** A missing reference is not a zero price. At the final pre-close snapshot, the bot's outside-reference total omitted 17 of 36 holdings. The analysis keeps cash, venue marks, and reference marks separate rather than turning missing data into a fictitious loss.

## How it was built and checked

Claude and Codex assisted with implementation, research, debugging, and analysis. Human decisions remained material: allocations, deployments, limits, exits, and manual trades. The project was an operating collaboration between software and its supervisor.

The retrospective draws on **24,064 leaderboard snapshots** and targeted extracts from a **10.9-million-row journal**. Independent checks reproduced the peak, drawdown, closing marks, unchanged final inventory, and MN-01 order arithmetic. The published data preserves measurement limits: complete realized profit and per-strategy P&L remain unreconciled, and final settlement is still pending.

The [full technical retrospective](docs/technical-retrospective.md) contains the operating history and detailed failure analysis. The [methods appendix](docs/methods.md) explains accounting, coverage, and source provenance. The [interactive report](https://tristandarnell.github.io/SIG-Predictions-Cup-Writeup/) presents the same observed performance with replay and election-night views.

*Presentation inspired by [IHateVolatility's SIG Prediction Cup write-up](https://github.com/IHateVolatility/SIG-Prediction-Cup). All results and figures here concern our own account.*
