# A-share Strong Stock Picker Skill

Codex skill for screening A-share momentum candidates with the local `a-share-finance` MCP server.

The strategy gates candidates by recent limit-up history, then ranks them with trend, moving averages, 200-session highs, K-line body structure, volume confirmation, and reduce/avoid risk flags. Outputs are research signals only, not trading instructions.

## What It Does

- Keeps only stocks that had a limit-up move in the last 20 trading sessions.
- Prefers stocks above MA5, with MA5 not turning down, and price above MA10/MA20.
- Highlights 200-session highs and strong "red fat, green thin" K-line structure.
- Flags volume-up price-stall, long upper shadow, bearish candles, failed repairs, and moving-average breaks.
- Supports `candidate`, `hold`, `reduce`, `watch_low_absorb`, and `avoid` interpretations.

## Requirements

- Codex with skill support.
- The local `a-share-finance` MCP server.
- Optional but recommended: a TickFlow API key configured outside the repo.

Do not commit real API keys. Configure TickFlow through environment variables:

```bash
export TICKFLOW_API_KEY="your-tickflow-api-key"
export TICKFLOW_BASE_URL="https://api.tickflow.org"
```

## Install

Clone this repository and copy the skill folder into your Codex skills directory:

```bash
git clone https://github.com/icekale/a-share-strong-stock-picker.git
mkdir -p ~/.codex/skills
cp -R a-share-strong-stock-picker/a-share-strong-stock-picker ~/.codex/skills/
```

Restart Codex after installing or updating the skill.

## MCP Setup

The skill expects an MCP server named `a-share-finance` with tools such as:

- `strong_stock_scan`
- `stock_strength`
- `recent_limit_up_candidates`
- `tickflow_realtime_quotes`
- `tickflow_klines`
- `tickflow_intraday_klines`
- `stock_universe`
- `stock_history`
- `stock_spot`

Example Codex config:

```toml
[mcp_servers.a-share-finance]
command = "/path/to/a-share-finance-mcp/.venv/bin/python"
args = ["/path/to/a-share-finance-mcp/server.py"]
startup_timeout_sec = 120

[mcp_servers.a-share-finance.env]
TICKFLOW_API_KEY = "your-tickflow-api-key"
TICKFLOW_BASE_URL = "https://api.tickflow.org"
```

Replace paths and API key values locally. Do not put real secrets in this repository.

## Usage

Ask Codex:

```text
扫描 A 股强势候选股，带早盘日内信号
```

or:

```text
分析 600000 是否符合强势股规则
```

The skill will instruct Codex to use `strong_stock_scan` for full-market scans and `stock_strength` for single-stock analysis.

## Data Path

Recommended path:

1. Use AKShare 东方财富涨停池 to build the recent 20-session limit-up candidate pool.
2. Use TickFlow realtime quotes, daily K-line, and intraday K-line to verify and rank candidates.
3. If a TickFlow batch endpoint is unavailable but the single-symbol endpoint works, the MCP should query symbols one by one and report the speed tradeoff.

## Safety

This skill is for research and trade planning. It must not present results as guaranteed profit, direct buy/sell orders, or a substitute for position sizing and risk control.
