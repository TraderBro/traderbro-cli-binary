---
name: traderbro
description: "Query analyst predictions, content, and market research from the TraderBro platform."
homepage: https://github.com/TraderBro/traderbro-cli-binary
metadata: {"clawdbot":{"emoji":"📊","requires":{"bins":["traderbro"],"env":["TRADERBRO_API_KEY"]},"install":[{"id":"brew","kind":"brew","formula":"traderbro/tap/traderbro","bins":["traderbro"],"label":"Install traderbro (brew)"}]}}
---

# TraderBro CLI

<!-- Source of truth: TraderBro/traderbro-cli:SKILL.md — auto-synced to traderbro-cli-binary on push to main. Edit it there, not in the binary repo. -->

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

> **Ranking analysts — there is NO "accuracy" metric.** If a user asks for the "most accurate", "best", or "highest hit-rate" analysts, rank by **return** — that is the number TraderBro uses to measure analyst quality (how much their calls actually made). Valid `--sort` values are `return`, `predictions`, `name`; anything else (e.g. `accuracy`) errors, so don't guess flags.
>
> To rank the analysts **on a specific symbol**, make ONE call: `traderbro symbol predictions <EXCHANGE:SYMBOL> --json` — it returns each analyst's calls and returns on that symbol, already aggregated. **Never page through the global `prediction list` to compute per-analyst or per-symbol stats yourself** — the analyst- and symbol-scoped endpoints already do that aggregation. Pulling the whole prediction table page by page is slow, wasteful, and unnecessary; if a metric isn't directly available, fall back to `return`, don't brute-force it.

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
# Returns EXCHANGE:SYMBOL values e.g. NASDAQ:TSLA — use that in insight/predictions

# Insight = analyst commentary/insight/prediction mentions, with sentiment + tags.
# Filters compose: --symbol / --analyst / --type / --tag / --sentiment (run `traderbro insight --describe`).
traderbro insight --symbol NASDAQ:TSLA --json
traderbro insight --symbol NASDAQ:TSLA --type evidence_insight --sentiment negative --json
traderbro insight --analyst ray-wang --type catalyst_insight --json

# Curated, return-tracked directional calls only:
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

## Screener

```bash
# Run a screen against today's DataMatrix
traderbro screener run \
  --filter "exchange:eq:NASDAQ" \
  --filter "market_cap:gt:5B" \
  --filter "dollar_volume:gt:200M" \
  --sort ch1m:desc \
  --limit 50 --json

# Just the symbol list, piping into another tool
traderbro screener run --filter "ch1m:lt:-20" --symbols-only \
  | traderbro calculated-events scan --type CDLHAMMER

# Schema discovery — `schema` ALWAYS returns an OBJECT, never a top-level array.
# Find field keys with --search <kw> (keys at .results[].key) or --group <category> (keys at .fields[].key):
traderbro screener schema --group growth --json | jq -r '.fields[].key'
traderbro screener schema --search rsi   --json | jq -r '.results[].key'

# Categorical values for a field
traderbro screener values sector --json
```

Filter ops: `gt, lt, gte, lte, eq, neq, between, in`. Value suffixes: `K, M, B, T`.

**Universe construction pattern** (used by the `tv-bulk-scanning` skill):
```bash
RUN_DIR=/tmp/scan-$(date +%s); mkdir -p "$RUN_DIR"
traderbro screener run --filter exchange:eq:NASDAQ --sort dollar_volume:desc \
  --limit 100 --symbols-only > "$RUN_DIR/universe.txt"
# universe.txt: 100 lines, no footer — footer goes to stderr
```

## Two charts — which to use

| Need | Command | Why |
|---|---|---|
| Proprietary overlays (analyst marks, calculated events, predictions) | **`brochart`** | only the traderbro.ai-hosted chart carries TraderBro's custom data |
| High-fidelity price/volume: intraday, extended-hours, long history, bulk scanning, native drawing | **`tvsandbox`** | drives the official tradingview.com/chart over CDP on a dedicated Chrome |
| Chart-**pattern detection** (H&S, etc.) | **`calculated-events`** | server-side detector; there is no geometry detector in the CLI |

## Chart Control — official tradingview.com (traderbro tvsandbox)

Drives `tradingview.com/chart` over CDP on a dedicated, app-owned Chrome (port 9333,
profile `~/.traderbro/chrome-profile`). **Run `traderbro skills show tvsandbox-setup`
first** — the login gate (`whoami`/`login`), port isolation, and the single-Chrome
**sequential-only** rule are mandatory operating knowledge.

- **Read/judge a symbol:** `tvsandbox bars` + `tvsandbox snap` → Read the PNG → call it (skill `tvsandbox-reading`). No detector — judgement is the agent's.
- **Read many charts:** get a candidate list from `screener`/`calculated-events`, then rank it with `tvsandbox metrics` and/or bulk-capture with `tvsandbox sweep` (skill `tvsandbox-scanning`). tvsandbox does not screen the market itself.
- **Annotate:** `tvsandbox draw <shape> --points …` renders any of ~90 native TV objects; `tvsandbox clear` removes them (skill `tvsandbox-drawing`).

## Chart Control (traderbro brochart)

Requires: `traderbro brochart serve` running, TraderBro chart page open in browser.

**Standard single-chart workflow:**
1. `traderbro brochart symbol NASDAQ:NVDA 1D`          — set chart
2. `traderbro brochart range 2025-01-01`               — set visible range
3. `traderbro brochart close && traderbro brochart screenshot -o /tmp/chart.png`  — see what's on the chart
4. `traderbro brochart bars --last 90 --json`          — get exact timestamps for drawing
5. `traderbro brochart draw *`                         — annotate
6. `traderbro brochart close && traderbro brochart screenshot -o /tmp/annotated.png`  — verify
7. `traderbro brochart save "name"`                    — persist

### Command surface (read commands)

| Command | Purpose | Readback |
|---|---|---|
| `brochart health` | 5-check diagnostic (bridge, chart, widget, UDF, memory) | exit 0/1 |
| `brochart charts` / `brochart use <id>` / `brochart connect <uuid>` | List, select, or attach to a chart tab | JSON |
| `brochart state [--json] [--full]` | Symbol, resolution, bar_spacing, studies, shapes, theme | JSON (long-key shape) |
| `brochart bars --last N [--json]` | Authoritative OHLCV: `{date, timestamp, open, high, low, close, volume}` | always |
| `brochart study list [--json] [--plain]` | All active studies (id, name, parent_id) | human table default |
| `brochart study values --last N [--include-volume] [--ids X,Y]` | Bars + every active study's computed series, time-aligned | always |
| `brochart draw list [--json] [--plain]` | All shapes (id, name=type, parent_id for labels) | human table default |
| `brochart saved [--json]` | Saved chart layouts | human table default |
| `brochart search <q> [--exchange] [--type] [--limit]` | UDF symbol search; falls back to `/cli/v1/symbols/search/` when no chart connected | always |
| `brochart screenshot [-o file]` | PNG. Base64 to stdout if no `-o`. | binary |

### Command surface (write commands — all return `{ok, command, ...fields}` under `--json`)

| Command | Effect | Notes |
|---|---|---|
| `brochart symbol <ticker> [res]` | Set symbol + optional resolution | Always exchange-qualify (`NASDAQ:AAPL`) to avoid bare-ticker warnings + cross-exchange resolution surprises. Emits stderr warning when symbol has no UDF data (chart loads empty); exit 0 — next `brochart symbol` recovers. |
| `brochart timeframe <val> [res]` | Set visible window: `1D 5D 1M 3M 6M YTD 1Y 3Y 5Y ALL`; optional res `1D/1W/1M` | Computes (from, to) Unix timestamps in Go and calls `setVisibleRange` — TV's documented `setTimeFrame` shape doesn't work in this Charting Library variant. |
| `brochart zoom <in\|out\|reset\|N>` | Bar spacing. Use `-- -5` to pass a negative value (cobra). | Verified via `brochart state \| jq .bar_spacing` |
| `brochart range <start> [end]` | Explicit YYYY-MM-DD window | No readback — visual diff only |
| `brochart close` | Dismiss popups/panels | Run before screenshot |
| `brochart study add <name> [--force]` | Add indicator. Use full names ("Relative Strength Index", not "RSI"). | Dedup by name unless `--force`. |
| `brochart study remove <id>` / `brochart study clear` | Remove one or all studies | |
| `brochart draw line/hline/rect/arrow/text/long/short` | Draw shapes. `--label` adds a linked text shape (parent_id). | `--json` returns `shape_id` for the new shape |
| `brochart draw modify <id> [--color] [--width] [--style] [--label]` | Edit existing shape's visual properties | Phase 1: color/width/style/label. Phase 2 (geometry) deferred. |
| `brochart draw remove <id>[,<id>] / --type X / --all` | Surgical shape removal with label cascade | `--all` is alias for `brochart draw clear` |
| `brochart draw clear` | Wipe all shapes | |
| `brochart save [name]` / `brochart refresh [--hard]` | Save layout / soft re-bind (default) / hard reload page | `--hard` does origin pre-probe and refuses if front-end down |
| `brochart eval "<js>"` | Escape hatch for un-wrapped TV API calls | Use sparingly |

### Key facts

- **Always run `brochart close` before a screenshot** to dismiss indicator panels.
- **Always run `brochart bars` before drawing** for authoritative timestamps — computed dates can land one bar off around weekends/DST.
- **Always exchange-qualify symbols** in scripts (`NASDAQ:AAPL`, not `AAPL`) — bare tickers print a stderr warning and may resolve to a wrong exchange after a saveload restore.
- **`brochart state --json`** has long keys: `.symbol` is a string (the ticker), `.symbol_info` carries the metadata, `.bar_spacing` reflects current zoom level.
- **`brochart bars --json`** keys: `open, high, low, close, volume, timestamp, date` (long form, same as `brochart study values`).
- **`brochart eval`** is an escape hatch only — if you use it twice for the same thing, it belongs in `chart-bridge.js`.
- **Soft refresh** (`brochart refresh`) is ~200 ms and non-destructive — fine for K=25-50 cadence in long loops. **Hard refresh** (`brochart refresh --hard`) is destructive and pre-probes the chart origin (refuses if unreachable); reserve for `brochart health` failures.
- **No-bars charts** show "No data here" — chart accepts next symbol normally, not a freeze. In production (traderbro.ai) the UDF covers every active US symbol so this usually means a wrong-exchange ticker (e.g. `BCBA:AAPL`); in local dev backends, even mega-caps may not be ingested yet. `brochart symbol` and `brochart draw` both emit stderr warnings + exit 0 on bars=0; suppress with `--quiet`.

### Bulk scanning (>20 symbols in one session)

Read `traderbro skills show tv-bulk-scanning` for the on-disk corpus pattern. The TL;DR:

```bash
RUN_DIR=/tmp/scan-$(date +%s); mkdir -p "$RUN_DIR"/{ss,bars,values}
traderbro screener run --filter exchange:eq:NASDAQ --sort dollar_volume:desc \
  --limit 100 --symbols-only > "$RUN_DIR/universe.txt"

K=0
while read sym; do
  traderbro brochart symbol "$sym" 1D --quiet >/dev/null
  sleep 1  # let bars load
  traderbro brochart bars --last 90 --json > "$RUN_DIR/bars/${sym//:/_}.json"
  K=$((K+1)); (( K % 25 == 0 )) && {
    traderbro brochart health --quick >/dev/null || traderbro brochart refresh
  }
done < "$RUN_DIR/universe.txt"

# Now analyse OFFLINE via jq across the corpus
for f in "$RUN_DIR"/bars/*.json; do
  jq -r --arg sym "$(basename "$f" .json)" \
    '[.bars[].close] as $c | "\($sym)\t\(($c|last - ($c|first)) / ($c|first) * 100)"' "$f"
done | sort -k2 -rn | head
```

### Visual regression for blind-write commands

`brochart timeframe`, `brochart range`, `brochart close`, `brochart screenshot` have no state readback. Use the helpers at `experiments/tv_visual_helpers.sh`:

```bash
source experiments/tv_visual_helpers.sh
verify_visual_change traderbro brochart timeframe 6M
verify_n_distinct_screenshots /tmp/sweep "traderbro brochart timeframe" 1D 6M 1Y ALL
```

These exist for testing only — don't run them inside production agent loops (each screenshot is ~100 KB / ~500 ms).

## Available skills

`traderbro skills list` returns these (sorted; run `traderbro skills show <name>` for details):

| Skill | When to use |
|---|---|
| `analyst-top-picks` | "what should I buy" / best analyst picks |
| `calculated-events-tv` | end-to-end signal scan + visual annotation across symbols |
| `chart-reading` | interpret a chart PNG (from `tvsandbox snap` or `brochart screenshot`) |
| `pattern-confluence` | Buy/Sell/Hold verdict combining analysts + signals + chart |
| `pattern-effectiveness` | "does this pattern actually work on this symbol" |
| `sharing` | producing shareable URLs + preview images for social/messaging |
| `signals-analysis` | run `calculated-events` detectors, optionally annotate |
| `symbol-stock-picker` | candidate discovery from trending coverage |
| `tv-analysis` | single-chart annotation SOP |
| `tv-bulk-scanning` | loop over many charts, dump artifacts, analyse offline |
| `tv-drawings` | TV's built-in shape vocabulary via `createMultipointShape` |
| `tv-long-running` | reliability primitives (`brochart health`, `brochart refresh`) for batch agents |
| `tv-patterns` | end-to-end chart-pattern analysis: detect + annotate |
| `tv-quant` | pull exact numeric values (OHLCV + study series) |
| `tvsandbox-setup` | **read first** for tvsandbox — login gate, port 9333, sequential-Chrome rule |
| `tvsandbox-reading` | read/judge one symbol on official tradingview.com data (bars + screenshot) |
| `tvsandbox-scanning` | rank/bulk-capture a symbol list (from screener/calculated-events) via `metrics`/`sweep` |
| `tvsandbox-drawing` | annotate with native TV objects via `tvsandbox draw` |

## Known issues + recent fixes

See `docs/16_TV_NewBugs_BUG25_BUG26.md` for the running bug log. Current state (2026-05-14):

- **BUG-25 fixed** — Volume study's `symbol` input no longer pins to the symbol at creation time; `brochart timeframe` with resolution change no longer corrupts study symbols.
- **BUG-26 fixed** — `brochart range` "Value is null" was a downstream symptom of BUG-25; gone.
- **`brochart timeframe` no-op fixed** — was silently doing nothing because of a wrong payload shape; now computes (from, to) in Go and uses `setVisibleRange`.
- **BUG-27 retracted** — wasn't a real freeze; was no-UDF-data symbols mishandled. Now a soft stderr warning.

Test artifacts:
- `experiments/run_300_regression.sh` — 300-prompt regression with visual asserts on TF/zoom/range. Run after any `cmd/tv/*` change. Expect 298 PASS / 0 FAIL / 2 SKIP.
- `experiments/run_rapid_fire_stress.sh` — 9-phase rapid-fire stress covering symbol/study/draw/zoom/TF/range churn + chaos. Use when changing the bridge or `brochart symbol` plumbing.
- `experiments/tv_visual_helpers.sh` — reusable visual-diff helpers.

## Notes

- Always use --json for agent/script use.
- Use --jq to filter results: traderbro analyst list --json --jq '.results[].name'
- Exit code 2 = auth failure; 3 = not found; 6 = quota exceeded; 0 = success.
- Pagination: use --limit and --page flags.
- Share commands (analyst/prediction/symbol share) are quota-free.
