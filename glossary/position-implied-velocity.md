# Position-Implied Velocity (PIV)

**Position-Implied Velocity is the rate at which positions are being added or removed on a prediction-market contract, derived from the count of 1¢ price-delta events recorded in the market_indicator_history table over a 7-day rolling window. PIV is the substrate metric that separates a contract that is being actively traded from one whose price moves come from a single posted quote and stale fills.**


## Explanation

## The Reason Raw Price Velocity Is Not Enough

A contract that moves from 32¢ to 35¢ once and sits there for a week looks the same on a daily candle as a contract that bounces between 32¢ and 35¢ a hundred times in the same period. The first one is a single posted quote that nobody contested. The second one is a market actively repricing on every print. PIV is the metric that separates them.

The substrate is the `market_indicator_history` table: every time the mid-price crosses a 1¢ threshold (32 → 33, 33 → 32, 33 → 34, etc.), the cron job logs a row with the timestamp. Computing PIV is then just counting rows in a rolling window:

```
PIV_7d = count(market_indicator_history rows where market_id = X and t > now − 7d)
```

The window choice (7 days) is deliberate. One day is too short — most non-event markets will look dead even when they are normal. Thirty days is too long — by the time PIV catches up, the live activity is already old news. Seven days catches most of what a daily scan needs and is the default in `sf scan --by-piv`.

## Why 1¢ Is the Threshold

There is nothing magic about 1¢ as a threshold. It is the rounding unit Kalshi uses to display prices, so any price move smaller than that is invisible at the API level anyway. Going to 0.5¢ would force us to track sub-cent fills (which Kalshi does support, but the noise overwhelms the signal). Going to 2¢ would deflate the count enough that small markets always look dead.

The 1¢ delta is a "minimum interesting move" filter. A market that crosses 1¢ thresholds twenty times in seven days is interesting. A market that crosses zero thresholds is either truly stable or completely empty. PIV tells you which.

## How to Read It

PIV is most useful as a *gate* combined with another indicator. A high IY with low PIV usually means stale data — nobody is trading the contract at the displayed price, so the IY is theoretical. A high IY with high PIV means a real, active dislocation that the market is currently chewing on. The combination is what makes PIV worth computing.

The other interesting read is *changes* in PIV over time. A market that was posting PIV = 3 for weeks and suddenly jumps to PIV = 25 in a day is reacting to something. Often that something arrives in the news flow about an hour later. PIV is one of the few metrics in the stack that occasionally leads, rather than lags, the headline.


## Example

Two Polymarket contracts on the same morning, both showing nearly identical IY:

| Contract | YES Price | τ remaining | IY | PIV (7d) |
|---|:---:|:---:|:---:|:---:|
| Election-Wisconsin-Senate | 0.34 | 88 days | 615% | 47 |
| Niche-Reg-Approval | 0.34 | 91 days | 605% | 2 |

The IY column says these are essentially the same trade. The PIV column says they are not. The first contract has 47 1¢-delta events in the last week — somebody is actively repricing it, every couple of hours, as polls and news arrive. The second contract has had 2 events in seven days, meaning the price moved twice and sat there. The 605% IY on contract two is theoretical because nobody is currently transacting at it.

If you put a 100-share order in on contract two, you are quite likely to be the only filled trade of the week, and your fill price could be very different from the displayed mid (because the resting orders may have been there since the price last moved). Contract one is where the IY is *executable*. Contract two is where the IY is *displayed*.

The composite read: trade contract one with normal sizing, and either quote contract two as a maker or skip it. The sf CLI surfaces this distinction with `sf scan --by-iy desc --min-piv 10`, which filters out the stale-IY contracts before they reach the human reviewer.


## CLI

```bash
sf scan --by-iy desc --min-piv 10
```


## Related

[liquidity-availability-score](liquidity-availability-score.md), [cliff-risk-index](cliff-risk-index.md), [implied-yield](implied-yield.md), [volume](volume.md), [orderbook](orderbook.md), [edge-detection](edge-detection.md), [null-as-signal](null-as-signal.md), [signal](signal.md)
