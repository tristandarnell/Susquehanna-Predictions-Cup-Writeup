# Project summary

## Short introduction

Built and operated a Rust/Tokio trading system for the SIG Predictions Cup, integrating Kalshi and Polymarket US reference data with contract mapping, execution, inventory management, and Python research tools. The account grew from 100,000 to 762,109.67 SUSQies in marked value (+662.11%) and stood sixth on the first post-close leaderboard. The venue recorded 27.04 million of volume across 18,027 orders with fills. Research included queue-sensitive replay and forward shadow testing; post-competition analysis reconciled leaderboard observations, order responses, and position snapshots to separate execution failures from holding risk. Results are provisional pending settlement. Development was AI-assisted and trading was human-supervised.

## Resume bullet options

Choose the bullets that best match the role and your own contribution; retain the qualification on performance.

- Built a Rust/Tokio prediction-market trading engine with multi-source price validation, streaming/REST reconciliation, inventory controls, and Python research tooling; operated through the SIG Predictions Cup with 27.04M SUSQies of venue-reported volume.
- Developed queue-sensitive replays and forward shadow experiments to evaluate trading ideas under partial fills, capital constraints, and adverse selection; rejected tighter market making when realistic queue assumptions reversed its simulated profitability.
- Grew a 100K-SUSQie account to 762K in marked value (+662.1%), standing sixth on the provisional post-close leaderboard; analyzed 24,064 leaderboard snapshots and order/position evidence to distinguish execution loss from subsequent inventory repricing.

## Technical discussion points

**For quantitative roles:** how a reference price differs from an executable hedge; why queue assumptions reverse simulated profitability; capital opportunity cost; race-level exposure aggregation; separating realized returns, marks, and settlement.

**For engineering roles:** asynchronous fills and unknown submission outcomes; deduplication across reporting paths; idempotent retries; account-wide reservations; reconciling manual activity; invalidating stale exit instructions after a position reversal.

**For a project walkthrough:** start with the strategy and architecture, show the full observed account curve, explain one rejected research idea, then walk through the MN-01 incident and its proposed state-transition fix. Describe Claude and Codex's development assistance alongside the human decisions and verification work.

The [full retrospective](../README.md) and [methods](methods.md) contain the supporting definitions and evidence limitations.
