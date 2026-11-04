# Project summary

I ran a Rust/Tokio trading bot in the SIG Predictions Cup. It compared SIG prices with Kalshi and Polymarket US, placed orders, and tracked fills and inventory. Python tools handled research and monitoring. Claude and Codex helped with development; I controlled allocations and also traded manually.

The account grew from 100,000 to **762,110 SUSQies in marked value (+662.1%)**, placing sixth at trading close. Results remain provisional until the open positions settle. Turnover was 27.04 million across 18,027 orders with fills.

The work covered contract matching, missed and duplicate fills, order retries, cash constraints, and queue assumptions in backtests. One market-making replay went from roughly +1,250 per day to −414 when the queue model became more realistic. That experiment stayed in shadow mode.

The [technical notes](technical-retrospective.md) explain the implementation and election-night mistakes. The [methods](methods.md) describe the calculations and data gaps.
