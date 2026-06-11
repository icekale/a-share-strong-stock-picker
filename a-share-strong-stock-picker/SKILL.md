---
name: a-share-strong-stock-picker
description: Use when Codex needs to screen A-share strong stocks, scan the whole A-share market, analyze stocks that had limit-up moves in the last 20 trading sessions, rank candidates by trend/K-line/volume strength, or explain candidate/hold/reduce/avoid/watch-low-absorb signals using the local a-share-finance MCP server with optional TickFlow realtime data.
---

# A-share Strong Stock Picker

## Overview

Use the local `a-share-finance` MCP server to run a disciplined A-share momentum scan. Treat outputs as research and trade-planning signals, not investment advice or automatic trade execution.

## Data Sources

Prefer the MCP server tools in this order:

1. `strong_stock_scan`: full-market or supplied-symbol scan. This is the primary tool.
2. `stock_strength`: single-stock analysis when the user names one stock.
3. `recent_limit_up_candidates`: use when inspecting the 20-session limit-up candidate pool.
4. `tickflow_realtime_quotes`, `tickflow_klines`, `tickflow_intraday_klines`: use for diagnostics or custom scans.
5. `stock_universe`, `stock_history`, `stock_spot`: use only when the primary tools are unavailable or the user asks for raw data.

TickFlow is the preferred paid provider. It requires `TICKFLOW_API_KEY` in the MCP server environment. The current practical scan path is: AKShare 东方财富涨停池 for the 20-session limit-up candidate gate, then TickFlow realtime, daily K-line, and intraday K-line for candidate verification. If TickFlow lacks a batch permission but has the single-symbol endpoint, the MCP will query symbols one by one; report the speed tradeoff rather than treating it as a data failure.

## Scan Workflow

For a full A-share scan:

1. Call `strong_stock_scan(provider="auto", universe_id="CN_Equity_A", max_symbols=0, result_limit=<user_limit>, include_intraday=<true when the user asks for盤中/早盘 rules>)`.
2. Expect the MCP to build a recent limit-up candidate pool first, then verify those candidates. Do not request raw full-market K-lines unless the user explicitly asks.
3. Sort and present by `status`, then `score`, then key positive signals.
4. Always include `risk_flags` for each name. Do not hide reduce/avoid signals behind a high score.

For a quick test or quota-conscious scan, use `max_symbols=300` before running the full universe.

For a user-supplied watchlist, pass `symbols=[...]` and do not scan the full universe.

## Strategy Rules

The scanner encodes these rules:

- First gate: only consider stocks with a limit-up move in the last 20 trading sessions.
- Trend first: prefer price above MA5, MA5 not turning down, and close above MA10/MA20.
- Strong K-line structure: red bodies should be larger than green bodies, with stronger volume on up days.
- Highest-quality signal: fresh or near 200-session high while still above short moving averages.
- Continue holding strength when volume confirms price progress.
- Reduce risk when volume expands but price stalls, when there is a long upper shadow, or when high-open strength fails to seal a limit-up.
- Avoid or empty-position when MA5 turns down, price closes below key moving averages, a solid bearish candle appears, or a broken limit-up has not repaired.
- Entry discipline: do not chase red candles; prefer green pullbacks that stay above MA5/MA10/MA20 or regain intraday averages.
- Exit discipline: do not sell only because the candle is green; reduce into red strength or failed breakout signals.
- No-trade discipline: if trend is unclear, say "空仓/观望" instead of forcing a pick.

## Output Format

Keep the answer compact and operational:

- Data source and limitations: TickFlow realtime, TickFlow daily only, or AKShare fallback.
- Top candidates table: symbol, Chinese status from `status_zh`, score, key positives, key risks. Do not show raw English `status` unless the user asks for API fields.
- Action interpretation:
  - `候选`: strong candidate, but entry still waits for non-chasing setup.
  - `持有/观察`: trend intact, keep observing/holding.
  - `减仓`: lock profit or reduce exposure.
  - `低吸观察`: possible low-absorb watch after sharp drop and recovery.
  - `空仓/回避`: empty-position/avoid.
- Final note: "趋势优先，信号不清就空仓。"

Never present the result as guaranteed profit, a direct order, or a substitute for position sizing and risk control.
