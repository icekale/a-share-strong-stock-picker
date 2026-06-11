# A 股强势选股 Skill

这是一个用于 Codex 的 A 股强势选股 skill，配合本地 `a-share-finance` MCP 使用。

它会先用“最近 20 个交易日是否涨停过”作为第一道门槛，再结合趋势、均线、200 日新高、K 线实体结构、量能确认、放量滞涨和空仓风险信号，对候选股进行排序和解释。输出仅用于研究和交易计划，不是买卖指令。

English version is available below: [English Version](#english-version).

## 功能

- 只保留最近 20 个交易日内有过涨停的股票。
- 优先选择股价在 MA5 上方、MA5 未拐头向下、并站上 MA10/MA20 的股票。
- 标记 200 日新高和“红肥绿瘦”的强势 K 线结构。
- 识别放量滞涨、长上影、实体阴线、断板未修复、跌破均线等风险。
- 输出 `candidate`、`hold`、`reduce`、`watch_low_absorb`、`avoid` 等状态解释。

## 依赖

- 支持 skills 的 Codex。
- 本地 `a-share-finance` MCP server。
- 推荐配置 TickFlow API key，用于更稳定的实时行情、K 线和日内数据。

不要把真实 API key 提交到仓库。TickFlow key 应通过本机环境变量或本机 Codex MCP 配置传入：

```bash
export TICKFLOW_API_KEY="your-tickflow-api-key"
export TICKFLOW_BASE_URL="https://api.tickflow.org"
```

## 安装

克隆仓库，并把 skill 文件夹复制到 Codex skills 目录：

```bash
git clone https://github.com/icekale/a-share-strong-stock-picker.git
mkdir -p ~/.codex/skills
cp -R a-share-strong-stock-picker/a-share-strong-stock-picker ~/.codex/skills/
```

安装或更新后，重启 Codex。

## MCP 配置

该 skill 需要一个名为 `a-share-finance` 的 MCP server，并依赖这些工具：

- `strong_stock_scan`
- `stock_strength`
- `recent_limit_up_candidates`
- `tickflow_realtime_quotes`
- `tickflow_klines`
- `tickflow_intraday_klines`
- `stock_universe`
- `stock_history`
- `stock_spot`

Codex 配置示例：

```toml
[mcp_servers.a-share-finance]
command = "/path/to/a-share-finance-mcp/.venv/bin/python"
args = ["/path/to/a-share-finance-mcp/server.py"]
startup_timeout_sec = 120

[mcp_servers.a-share-finance.env]
TICKFLOW_API_KEY = "your-tickflow-api-key"
TICKFLOW_BASE_URL = "https://api.tickflow.org"
```

请在本机替换路径和 API key。不要把真实密钥放进这个仓库。

## 使用方式

可以直接问 Codex：

```text
扫描 A 股强势候选股，带早盘日内信号
```

也可以分析单只股票：

```text
分析 600000 是否符合强势股规则
```

skill 会引导 Codex 使用 `strong_stock_scan` 做全市场或股票池扫描，使用 `stock_strength` 做单股分析。

## 数据路径

推荐路径：

1. 用 AKShare 东方财富涨停池构建最近 20 个交易日涨停候选池。
2. 用 TickFlow 实时行情、日 K 和日内 K 复核并排序候选股。
3. 如果 TickFlow 某个批量接口不可用，但单标的接口可用，MCP 应逐只查询并说明速度取舍。

## 安全边界

这个 skill 只用于研究和交易计划。它不能把结果表述为确定盈利、直接买卖指令，也不能替代仓位管理和风险控制。

## English Version

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
