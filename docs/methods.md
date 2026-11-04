# Methods and evidence

This appendix accompanies the November 4, 2026 pre-settlement retrospective. All currency amounts are competition SUSQies. No settled finish or cash withdrawal is claimed.

## Source coverage

The analysis uses copies retrieved from the operating server through read-only access. Two low-priority sequential scans used the same frozen journal boundary. The full 7.42 GB journal was scanned remotely; relevant rows were downloaded, rather than representing the extracted subset as the complete source file.

| Evidence | Coverage |
|---|---|
| Leaderboard | 24,064 snapshots, October 1 at 23:26:57 EDT through November 4 at 15:37:44 EST |
| Frozen live journal | 10,922,426 source rows, October 1 at 15:38:14 UTC through November 4 at 20:39:15 UTC |
| First journal extraction | 118,959 selected source rows, plus public-trade aggregates and extraction metadata |
| Second extraction | 187,429 selected order/control rows, including all 177,466 `batch` and 369 `batch_unknown` rows |
| Status and position series | 9,847 selected status observations and 8,305 position snapshots |
| Supporting records | Running executable identity, current source/configuration snapshot, selected prior source snapshots, operator history and research artifacts |

The frozen journal's complete-line SHA-256 is `bedd35137a8bcc820a98ae843a41274c4abf3057aa6008e636beda457f35f0e8`. The downloaded leaderboard archive SHA-256 is `e55d1cc3aa2864b43008007db52fb8d0f56942cc3f2fb4f54fdeaa769ffbc301`.

Private evidence includes raw order responses, source snapshots and manifests. It is deliberately outside this bundle. The public datasets include selected account observations and calculated metrics, without infrastructure addresses, credentials, raw configuration, or counterparties' account records. The bundle is a retrospective and supporting data, not a runnable trading system.

The strategy description was checked against the October 3 strategy brief, archived configuration, deployment logs, source code, and runtime sweep events. Quoted ladder settings describe specific versions, not one configuration used throughout the month. The October 5 incident review supersedes the earlier thin-book diagnosis. Neither reference-price edge nor overlapping strategy-event counts are treated as realized strategy profit.

## Leaderboard accounting

For this tournament, the account series is reconstructed as:

```
marked account value = 100,000 initial capital + leaderboard pnl
marked return = (marked account value - 100,000) / 100,000
drawdown at time t = prior observed maximum account value - account value at t
drawdown fraction = drawdown / corresponding prior observed maximum
```

This follows the saved leaderboard schema and the system's contemporaneous leaderboard/portfolio reconciliation. The exact proprietary mark calculation is not reverse-engineered here. In particular, this analysis does not assume an unverified four- or five-hour VWAP formula.

The maximum observed drawdown is 337,464.07 from 1,054,548.74 on October 30 at 16:18:32 EDT to 717,084.67 on November 4 at 11:24:42 EST, or 32.0008%.

The first post-close mark is 13,344.40 above the last pre-close observation despite unchanged reported volume and filled-order count. It is labeled by its actual observation time. The latest saved snapshot repeats that post-close mark.

The first 11.45 hours after the scheduled opening have no leaderboard observations. The initial 100,000 allocation is an isolated point in the chart. Four later gaps exceed ten minutes; the longest is about seventeen minutes. The figure breaks sufficiently long gaps instead of treating them as measured continuous paths. “Best observed rank” does not assert that a better rank was impossible in an unobserved period. Top-N snapshots do not establish the total number of entrants.

Daily observations use the final saved snapshot within each local calendar day. The opening-day change is relative to the known initial allocation, despite missing intraday coverage. November 4 is a partial competition day and includes the frozen post-close mark. Daily changes are changes in marked account value, not daily realized trading income.

The [official portfolio documentation](https://sig.thesuper.market/docs/portfolio-and-leaderboards) describes portfolio and leaderboard displays. The [settlement rules](https://sig.thesuper.market/docs/settlement-and-payouts) state that final standings follow settlement of every market. All 237 captured market-status responses in the closing control records were `closed`, not `settled`; no independent final settlement ledger was retrieved.

## Cash, holdings, and outside-reference valuation

The cash series is sampled bot telemetry. Remaining acquisition basis is the cost assigned to inventory still held, not cumulative spending or portfolio value. Early telemetry includes transient accounting anomalies, so the cash/basis figure focuses on November 1–4; it does not clip early spikes and present the result as a full-month series.

For a holding with signed YES quantity `q` and a usable contemporaneous YES reference `f`, the reference holding value is:

```
q >= 0: q * f
q < 0: abs(q) * (1 - f)
```

The remaining-holding comparison uses the venue's recorded entry average, rounded to four decimals, for acquisition basis. A race-level group adds contracts pointing to the same outcome. The CO-03, Mississippi Senate, and MN-01 figures cover only the remaining holdings in those three groups. Earlier exits, other markets, and any charges not incorporated in entry averages are excluded.

At the final pre-close snapshot, 17 of 36 holdings lacked valid reference prices, covering 576,167 contracts and approximately 444,516.82 of remaining entry basis. The bot's `value_at_fair` omitted those positions. After close, all references were absent. Accordingly, neither the pre-close 265,930 nor the post-close zero is used as full-portfolio NAV. No outside-reference drawdown is calculated across changing coverage.

## Separating execution losses from carried exposure

The last raw execution was November 4 at 00:47:22 EST. At 00:47:49, the final filled-order-count update showed marked account value of 1,046,386.23. Reported cumulative volume and filled-order count then stayed fixed. Position quantities and entry averages were unchanged across 178 consecutive position snapshots from 00:51:40 through 15:36:43 EST.

The subsequent marked change to the first post-close value is:

```
762,109.67 - 1,046,386.23 = -284,276.56
```

This supports repricing of carried holdings as the explanation for that later interval, rather than additional executions. The remaining-holding reference losses are a separate measurement; they are not forced to sum to the venue's marked change.

For the earlier MN-01 incident, the position transition was:

```
-519 + 77,519 - 25,000 - 25,000 = 27,000 signed YES contracts
```

Authoritative responses record 25,000 YES sold in each of the two automated orders, with gross proceeds of 4,731.17 and 1,579.305. Both sold from the newly acquired long at a 0.645 matched purchase basis:

```
gross proceeds = 4,731.17 + 1,579.305 = 6,310.475
matched basis = 50,000 * 0.645 = 32,250
gross matched-basis P&L = -25,939.525
```

The displayed loss rounds to 25,939.53. This is a specific matched-lot calculation before any unshown charges, not the account's total realized P&L. A prior operator estimate near 31,400 compared the exits with a 0.75 fair price; it measured a different quantity and is not used as realized loss. Individual response `fillPrice` fields also differ from order-average execution prices; gross proceeds divided by filled quantity provide the relevant averages here.

## Fill reconstruction and its limits

The raw extraction contains 35,483 fill notifications referencing 17,220 distinct order IDs. Notifications overlap across REST, WebSocket, and strategy annotations. There are 425 exact repeated fingerprints before normalizing the different reporting paths. Neither deduplicating only exact rows nor adding every strategy-specific fill record yields a reliable execution ledger.

Joining authoritative API action/side fields to the order IDs resolves direction for **15,357 of 17,220 order IDs, or 89.18%**. That covers 30,931 raw notifications. The remaining 1,863 IDs retain unknown direction. No direction is inferred from whether the fill price was above or below fair; doing so would build presumed profitability into the classification.

For the matched orders, planning flags include passive orders, inventory exits, and taker orders. A passive flag does not establish profitable market making, and a flag is not a complete strategy identifier. Therefore this write-up does not publish a per-engine P&L table or a fabricated full-month realized/unrealized split.

The venue's 18,027 filled-order count and 27,044,840.85 volume are retained as venue-reported aggregates. They have not been forced to equal an incomplete reconstructed ledger. Research replay and shadow results are labeled as such and are not added to account performance.

## Architecture and historical attribution

The running executable hash was read independently. Current source files were retrieved with hashes and modification times consistent with the documented last deployment. That does not cryptographically bind the current disk source to the executable, nor reconstruct every earlier runtime configuration.

Architecture descriptions combine inspected source with dated operational evidence. A module present in source, an enabled configuration, a dispatched order, a filled order, and a profitable trade are different states. The C8 section uses the absence of recorded jump-dispatch events rather than asserting that a general order cap disabled a separately sized path.

Historical market-making numbers come from recorded October replay and forward-shadow work. They were not rerun as part of this retrospective and are not represented as live returns. The November 4 calculations were recomputed from retrieved observations, with independent checks of peak, drawdown, ranks, closing marks, unchanged holdings, and the MN-01 order responses.

## Files in this bundle

- `README.md`: the concise GitHub retrospective.
- `docs/technical-retrospective.md`: the full operating history and analysis.
- `report.html`: a recruiter-oriented visual case study with an interactive account timeline.
- `figures/`: four charts in PNG and SVG formats, plus an animated account replay.
- `data/metrics.json`: selected metrics with measurement definitions.
- `data/account-observations.csv`: the full saved own-account leaderboard series.
- `data/daily-observations.csv`: daily final observations and marked changes.
- `data/cash-and-basis-observations.csv`: selected final-four-day telemetry underlying the cash chart.
- `docs/methods.md`: this appendix.
- `docs/project-summary.md`: a short project introduction and resume bullet options.

All timestamps are retained in UTC and/or their original `America/New_York` offset. No fixed UTC offset is applied across the November 1 daylight-saving change.

Animation is a presentation of the recorded series, not an additional observation or a model of future returns. The observed decline remains in the full account curve. Performance labels retain their marked, pre-settlement basis in both the technical and recruiter-oriented versions.
