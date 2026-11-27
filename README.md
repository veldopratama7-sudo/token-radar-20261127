# Token Horizon

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)

Token Horizon — a token launch radar: **197 pre-token protocols** ranked by implied value, each
with a seven-step farm plan. Every protocol on the snapshot's 40-protocol watchlist is covered
(40/40), plus 157 early finds beyond the floor. Protocols whose token already trades are never
ranked: they are the peer evidence, shown on the Already trading view with their market caps.

An entry for the [make.fun token launch radar bounty](https://make.fun/bounty/token-radar-01).

## Run it

```sh
node serve.ts        # serves out/ at http://localhost:8080
```

Build the output first:

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../token-radar-01-kit/input/snapshot.json
```

Then open http://localhost:8080/dashboard/ in a browser. No build step, no network requests —
the dashboard is plain static files under `dashboard/`, and its data is `data.js` beside it.

## The model (horizon-1)

Implied value follows only from the snapshot's own figures, through a peer multiple computed
from the snapshot on every run. No protocol name is special-cased.

```
peerMultiple(category) = clip( (n * median(mcap/TVL of the category's tokenized protocols)
                               + 4 * globalMedian) / (n + 4), 0.02, 1.5 )
growth     = clamp(change_7d, -50, 100) per cent (null change counts as 0)
growthMult = 1 + (growth / 100) * 0.5
maturity   = 0.6 + min(age_days, 1095) / 1095 * 0.4   (age unknown: 365 days)
breadth    = 1 + 0.03 * min(chains - 1, 5)
implied    = max(1, round(tvl_usd * peerMultiple * growthMult * maturity * breadth))
```

Each protocol's mcap/TVL ratio is clipped to [0.005, 5] before the median, so one wild ratio
cannot move a category; a category with few tokenized peers is shrunk toward the global median,
and a category with no tokenized peers uses the global median. Age is the time since DefiLlama
first listed the protocol, the best record of how long it has run with no token.

### Calibration, computed from the sealed snapshot (2026-11-14, 758 protocols)

Global median mcap/TVL across the 144 tokenized protocols with a market cap and TVL above
zero: **0.1689x**.

| Category | Peers | Median mcap/TVL | Multiple used |
| --- | --- | --- | --- |
| Collateral Management | 1 | 5.0000x | 1.1351x |
| Prediction Market | 1 | 5.0000x | 1.1351x |
| Token Locker | 2 | 2.5326x | 0.9568x |
| Derivatives | 2 | 2.1619x | 0.8332x |
| Chain | 1 | 3.0264x | 0.7404x |
| Risk Curators | 1 | 2.9747x | 0.7301x |
| Cross Chain Bridge | 4 | 1.2205x | 0.6947x |
| Canonical Bridge | 8 | 0.7464x | 0.5539x |
| Decentralized AI | 1 | 2.0604x | 0.5472x |
| NFT Marketplace | 1 | 2.0017x | 0.5355x |
| Privacy | 3 | 0.9691x | 0.5119x |
| Options | 1 | 1.5145x | 0.4380x |
| Basis Trading | 8 | 0.4822x | 0.3778x |
| AI Agents | 2 | 0.6047x | 0.3142x |
| Dexs | 11 | 0.3424x | 0.2962x |
| RWA | 9 | 0.2748x | 0.2422x |
| Liquid Restaking | 2 | 0.1828x | 0.1735x |
| Payments | 2 | 0.1813x | 0.1730x |
| Leveraged Farming | 1 | 0.1725x | 0.1696x |
| CDP | 3 | 0.1687x | 0.1688x |
| Launchpad | 1 | 0.1306x | 0.1612x |
| Indexes | 3 | 0.1477x | 0.1598x |
| DOR | 1 | 0.0898x | 0.1531x |
| Lending | 15 | 0.1324x | 0.1401x |
| Liquidity Manager | 1 | 0.0150x | 0.1381x |
| Farm | 1 | 0.0050x | 0.1361x |
| Governance Incentives | 1 | 0.0050x | 0.1361x |
| Yield | 13 | 0.1206x | 0.1320x |
| Restaking | 2 | 0.0246x | 0.1208x |
| Dual-Token Stablecoin | 2 | 0.0183x | 0.1187x |
| Bridge | 8 | 0.0884x | 0.1152x |
| Staking Pool | 2 | 0.0050x | 0.1143x |
| Yield Aggregator | 5 | 0.0504x | 0.1031x |
| Onchain Capital Allocator | 3 | 0.0059x | 0.0990x |
| CEX | 11 | 0.0620x | 0.0905x |
| Liquid Staking | 11 | 0.0343x | 0.0702x |

### What the board ranks

A protocol is ranked when it records no token (no market cap, no CoinGecko id, no symbol) and
is not a wrapped, staked or bridged product of a token that already trades: its category is not
Bridge, Canonical Bridge, Cross Chain Bridge, Liquid Staking, Liquid Restaking, Restaked BTC or
CEX, its name carries no whole word wrapped, staked or staking, ends in no version marker, and
begins with no exchange's name. That is the watchlist's own rule minus the $50M floor, so every
watchlist slug passes it. The board holds all 40 watchlist protocols and every other protocol the
rule keeps: 197 rows.

Top of the board: steakhouse-financial $1,826M, sentora-curator $1,662M, gauntlet $1,087M
(all Risk Curators, $1.5–2.8B TVL, peer multiple 0.7301x from a single tokenized peer —
shown on each detail view, not hidden).

## Farm plans

Seven steps per protocol, derived from its own record — category, chains, TVL, age — never from
a list of names:

1. Fresh wallet, seed phrase offline.
2. The main position: the category's action (supply and borrow, provide liquidity, stake,
   margin, vault deposit, mint, cover) with a capital tier from TVL ($50 / $100 / $150 / $250)
   on its first chain, with a gas estimate for that chain.
3. A second move: a smaller position on its second chain, or its free points/quests tasks when
   it runs on one chain.
4. One qualifying action each week.
5. Hold 30 / 45 / 60 days by age, so the activity is counted in more than one snapshot.
6. Claim only from the link the protocol itself publishes; never pay to claim.
7. A farm log, then an unwind in two steps.

## Caveats

The board says a protocol records no token, never that it has none: a protocol on the list may
still have a token its DefiLlama record does not carry. Implied value is the model's claim
about a launch that may never happen, not a price. Snapshot: DefiLlama protocols endpoint,
fetched 2026-11-14T01:18:33Z; the data does not move.

## License

MIT — see LICENSE.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
