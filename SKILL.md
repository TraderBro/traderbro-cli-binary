---
name: traderbro
description: "Query analyst predictions, content, and market research from the TraderBro platform."
homepage: https://github.com/TraderBro/traderbro-cli-binary
metadata: {"clawdbot":{"emoji":"📊","requires":{"bins":["traderbro"],"env":["TRADERBRO_API_KEY"]},"install":[{"id":"brew","kind":"brew","formula":"traderbro/tap/traderbro","bins":["traderbro"],"label":"Install traderbro (brew)"}]}}
---

# TraderBro CLI

Query analyst predictions, content, and market research.

## Setup

1. Log in at https://traderbro.ai
2. Go to Settings → API Keys → Generate New Key
3. Run: traderbro configure --server https://traderbro.ai --key tb_sk_...

For agents, use env vars instead:
```
export TRADERBRO_SERVER="https://traderbro.ai"
export TRADERBRO_API_KEY="tb_sk_..."
```

## Discover capabilities

```
traderbro describe --json
```

## Discover workflow skills

Skills are step-by-step workflows that tell you how to chain CLI commands for common tasks (e.g. "what stocks should I buy?"). Check skills before answering any question about what to buy, analyse, or research.

```
traderbro skills list --json
```

Read the full instructions for a specific skill:

```
traderbro skills show <name>
```

Always run `traderbro skills list --json` first when the user's request matches a skill's `trigger_keywords`.

## Verify auth

```
traderbro whoami --json
```

## Collaborating with Humans

TraderBro has two interfaces:
- **CLI** — your interface. Use it to query, filter, and analyse data.
- **Web frontend** — the human's interface. Rich profiles, charts, comparison views.

**Always give humans a URL, not raw data.**

When you surface an analyst, prediction, or symbol to a human:
1. Run `traderbro analyst share <slug> --json` (or `prediction share`, `symbol share`)
2. Include `profile_url` so the human can explore on the web
3. Attach `og_image_url` when posting to Slack, Telegram, or Twitter — renders as a preview card
4. Use `suggested_caption` as a starting point for social posts

**When to use `share` vs `get`:**
- `analyst get` / `prediction get` — when YOU need the data for further analysis
- `analyst share` / `prediction share` — when you are surfacing a RESULT to a HUMAN
- Most final-answer responses to a human should include a `profile_url`

**Example agent reasoning:**
> User: "Who is the best analyst covering TSLA?"
> Agent: runs `analyst list --sector Technology --sort return --limit 3`
> Agent: runs `analyst share <top-slug>` to get profile_url and caption
> Agent: responds with stats + "See their full track record: {profile_url}"

## Analysts

```
traderbro analyst list --sort return --limit 10 --json
traderbro analyst get cathie-wood --json
traderbro analyst predictions cathie-wood --json
```

## Predictions

```
traderbro prediction list --symbol TSLA --json
traderbro prediction list --direction bullish --since 2025-01-01 --json
traderbro prediction get 42 --json
```

## Symbols

```bash
# If you have a ticker but need the exchange, search first:
traderbro symbol search "Tesla" --json
traderbro symbol search AAPL --json
# Returns EXCHANGE:SYMBOL values e.g. NASDAQ:TSLA — use that in mentions/predictions

# Mentions and predictions — use EXCHANGE:SYMBOL format
traderbro symbol mentions NASDAQ:TSLA --json
traderbro symbol predictions NASDAQ:TSLA --json
traderbro symbol predictions DSE:ABBANK --json
```

## Trending Symbols

```bash
# Most-covered symbols in the last 7 days
traderbro symbol trending --since 7d --json

# Most bullish NASDAQ stocks this month
traderbro symbol trending --since 1m --exchange NASDAQ --sort bullish --json

# Technology sector, all time
traderbro symbol trending --sector Technology --json

# Pipe to jq
traderbro symbol trending --since 7d --json | jq '.results[:5] | .[].ticker'
```

## Content

```
traderbro content list --analyst cathie-wood --source twitter --limit 5 --json
traderbro content get 123 --json
```

## Research

```
traderbro research list --category stock --country us --json
traderbro research get <slug> --json
# Get a real slug from: traderbro research list --json
```

## Analyst Analytics (Plans 63–65)

### Threshold filters on list
```bash
# Analysts with ≥15 predictions, sorted by return
traderbro analyst list --min-predictions 15 --sort return

# Positive lifetime return, ≥10 predictions
traderbro analyst list --min-predictions 10 --min-return 0.01
```

### Period-specific returns
```bash
# Best analysts by 3-month return
traderbro analyst list --period 3m --sort return --limit 10

# Show 1-month returns for NASDAQ analysts
traderbro analyst list --exchange NASDAQ --period 1m
```

### Sector/industry filtering on analyst list
```bash
# Top analysts covering Technology
traderbro analyst list --sector Technology --sort return

# Analysts covering Semiconductors with 5+ predictions
traderbro analyst list --industry Semiconductors --min-predictions 5

# JSON for agent use
traderbro analyst list --sector Financials --json | jq '.results[] | {slug, avg_return_in_sector}'
```

### Sector edge (per-analyst breakdown)
```bash
# Which sectors does an analyst excel in? (3-month returns)
traderbro analyst sector-edge crux_capital --period 3m

# Industry breakdown, JSON for agent use
traderbro analyst sector-edge aleabitoreddit --group-by industry --json

# Only segments with 5+ calls
traderbro analyst sector-edge crux_capital --min-calls 5
```

### Global sector map (cross-analyst)
```bash
# Which sectors are analysts most accurate in overall?
traderbro analyst sector-map

# Industry level, 3-month returns
traderbro analyst sector-map --level industry --period 3m

# Predictions made in Q1 2026 only
traderbro analyst sector-map --date-from 2026-01-01 --date-to 2026-03-31

# JSON for agent — top 5 sectors by return last month
traderbro analyst sector-map --date-from 2026-01-01 --date-to 2026-03-31 --period 1m --json | jq '.rows | sort_by(-.avg_return) | .[0:5]'
```

### Sector discovery
```bash
# What sectors are available?
traderbro sectors list

# What industries exist under Technology?
traderbro sectors industries Technology

# Pipe into analyst list (interactive)
traderbro analyst list --sector "$(traderbro sectors list | fzf)"
```

## Share Links (for sending to users or posting to social media)

```bash
# These endpoints are quota-free — call them as often as needed.

# Get a shareable link + preview image + ready-to-use caption:
traderbro analyst share cathie-wood --json
traderbro prediction share 1234 --json
traderbro symbol share NASDAQ:TSLA --json

# The response includes:
#   profile_url       — send this to the human so they can explore on TraderBro
#   og_image_url      — attach this image when posting to Twitter/Telegram/Slack
#   suggested_caption — use or adapt this as the post text

# Browse page URLs are in every list response envelope:
traderbro analyst list --json | jq '.browse_url'
traderbro prediction list --json | jq '.browse_url'
traderbro symbol trending --json | jq '.browse_url'
traderbro analyst sector-map --json | jq '.browse_url'

# profile_url is on every row in list responses:
traderbro analyst list --json | jq '.results[0].profile_url'
traderbro prediction list --json | jq '.results[0].profile_url'
```

## Liquidity gate — always filter the universe before scanning

The calculated-events store covers every active US symbol, including illiquid nano-caps and penny stocks. The same detectors that flag a `golden_cross` on Apple will also flag one on a $0.10 ticker that ran 20× on a single news event. Mathematically the cross is real; economically the signal is noise.

**Always upstream-filter with the screener** before piping into `calculated-events scan`. The standard liquidity gate for tradeable mega-cap signals:

```bash
traderbro screener run \
    --filter "exchange:eq:NASDAQ" \
    --filter "market_cap:gt:5B" \
    --filter "dollar_volume:gt:200M" \
    --symbols-only \
  | traderbro calculated-events scan --type bull_flag --within-days 7
```

Tighter gate (most liquid only): `market_cap:gt:50B` + `dollar_volume:gt:500M`.
Looser gate (include mid-caps): `market_cap:gt:2B` + `dollar_volume:gt:50M`.

**Calling `scan` without an upstream symbol filter scans the entire universe — penny stocks included.** This is rarely what an agent wants. If you genuinely want the full universe (e.g. computing aggregate counts for a market-internals dashboard), be explicit about it in your reasoning.

Concrete example of what happens without a gate: a top-of-the-day scan once surfaced NASDAQ:AIXI as a triple-confluence long (`golden_cross + yearly_high_break + inverse_h&s`). AIXI had opened at $12.59 the prior day from a prior close of $0.61 — a 20× moonshot. The detector was correct but the signal was untradeable. The liquidity gate above would have excluded it.

## Pattern effectiveness — "does this pattern work on this symbol?"

`calculated-events scan` answers *"which symbols just fired pattern X?"* but says nothing about whether the pattern *works* on those symbols. For that, use:

```bash
traderbro calculated-events patterns EXCHANGE:SYMBOL [flags]
```

It returns one row per event type with: full-history count, last occurrence, direction, and the **average forward return at 7 horizons** (1D/3D/7D/1M/3M/6M/1Y) along with sample size. Bearish returns are sign-flipped server-side — for both directions, a higher positive number means "the pattern played out as expected."

Use this for any quantified-effectiveness question:

```bash
# Does Bull Flag work on TSLA?
traderbro calculated-events patterns NASDAQ:TSLA --event-type bull_flag --json

# Top 10 best-performing patterns on AAPL at 3M horizon
traderbro calculated-events patterns NASDAQ:AAPL --top 10 --json

# Currently-relevant high-accuracy setups (above-direction-avg + recent + n>=3)
traderbro calculated-events patterns NASDAQ:NVDA --hot-only --horizon 1m --json

# Bullish-tagged patterns that actually drop the stock (contrarian setups)
traderbro calculated-events patterns NASDAQ:META --direction bullish --sort-by avg_return --asc --top 5
```

**Always quote `sample_size` when citing an average.** An n=1 "+85% avg" is honest but not statistically meaningful. The `--hot-only` filter requires n ≥ 3 by definition; for other uses, mention `n` to the user.

Neutral event types (`bb_squeeze`, `volume_spike`) return `returns: null` — no forward-return data by design. Don't quote returns for them.

See the `pattern-effectiveness` skill for full workflow guidance, including the sign convention with a worked example.

## Chart & Technical Analysis

### Full analysis (chart + patterns + signals)

```bash
traderbro analyze EXCHANGE:SYMBOL [flags]
```

Generates a candlestick chart with indicator overlays, scans for candlestick patterns,
detects structural chart patterns, and returns a combined signals summary.

```bash
# Quick analysis — default indicators (sma_50, sma_200, rsi), 6-month daily chart
traderbro analyze DSE:ACI

# 1-year chart with MACD added
traderbro analyze NASDAQ:AAPL --period 1y --indicators sma_50,sma_200,rsi,macd

# Bollinger Bands instead of moving averages
traderbro analyze DSE:ACI --indicators bbands,rsi

# Faster — skip structural pattern detection
traderbro analyze DSE:ACI --no-chart-patterns

# Raw JSON — use when you need to parse or pipe the output
traderbro analyze DSE:ACI --json
```

**Reading the chart image:**
The command saves the PNG to a local file and prints the absolute path to stdout.
Read that file using your image tool to see the chart visually.

```
# stdout example:
/home/user/.traderbro/charts/ACI_DSE_1d_6mo_analyze_20260508-120000.png

# Read it:
<use your Read/image tool on that exact path>
```

**Flags:**

| Flag | Default | Options |
|---|---|---|
| `--period` | `6mo` | `1mo`, `3mo`, `6mo`, `1y`, `2y` |
| `--interval` | `1d` | `1d`, `1wk` |
| `--indicators` | `sma_50,sma_200,rsi` | `sma_20`, `sma_50`, `sma_200`, `ema_20`, `ema_50`, `bbands`, `rsi`, `macd` |
| `--lookback` | `10` | Any positive integer — candle pattern scan window (bars) |
| `--no-chart-patterns` | off | Skip structural pattern detection (faster) |
| `--no-candle-patterns` | off | Skip TA-Lib candlestick scan |
| `--json` | off | Print raw JSON to stdout instead of saving PNG |

**Key fields in `--json` output:**

| Field | What it tells you |
|---|---|
| `candle_patterns[*].bars_ago` | 0 = today, 1 = yesterday. Patterns ≤ 2 bars ago are most actionable |
| `chart_patterns[*].status` | `"active"` = pattern still in play; `"historical"` = already resolved |
| `chart_patterns[*].target_price` | Pattern's measured price target |
| `chart_patterns[*].stop_price` | Suggested invalidation level |
| `signals.confluence_score` | 0–10. ≥7 = high conviction. ≤3 = mixed or no signal |
| `signals.dominant_direction` | Overall bias: `"bullish"`, `"bearish"`, or `"neutral"` |
| `signals.summary` | Plain-English summary — read this first before the raw arrays |

### Candlestick pattern scan only

```bash
traderbro patterns candles EXCHANGE:SYMBOL [flags]

# Last 10 bars (default)
traderbro patterns candles DSE:ACI

# Extend the scan window
traderbro patterns candles NASDAQ:TSLA --lookback 20

# JSON for parsing
traderbro patterns candles DSE:ACI --lookback 14 --json
```

Returns which TA-Lib patterns fired in the last N bars with direction and recency.
Patterns with `bars_ago` of 0 or 1 carry the most weight.

## Chart Control (traderbro tv)

Requires: `traderbro tv serve` running, TraderBro chart page open in browser.

**Standard workflow:**
1. `traderbro tv symbol NASDAQ:NVDA 1D`          — set chart
2. `traderbro tv range 2025-01-01`               — set visible range
3. `traderbro tv close && traderbro tv screenshot -o /tmp/chart.png`  — see what's on the chart
4. `traderbro tv bars --last 90 --json`          — get exact timestamps for drawing
5. `traderbro tv draw *`                         — annotate
6. `traderbro tv close && traderbro tv screenshot -o /tmp/annotated.png`  — verify
7. `traderbro tv save "name"`                    — persist

**Key facts:**
- Always run `tv close` before a screenshot to dismiss indicator panels
- Always run `tv bars` before drawing to get authoritative timestamps — computed
  timestamps can land one bar off around weekends and DST transitions
- `tv draw` commands leave the chart interactive (exitDrawMode fix applied)
- `tv state --json` tells you what symbol/resolution is currently displayed
- `tv eval` is an escape hatch only — if you use it twice for the same thing, it belongs in chart-bridge.js

## Notes

- Always use --json for agent/script use.
- Use --jq to filter results: traderbro analyst list --json --jq '.results[].name'
- Exit code 2 = auth failure; 3 = not found; 6 = quota exceeded; 0 = success.
- Pagination: use --limit and --page flags.
- Share commands (analyst/prediction/symbol share) are quota-free.
