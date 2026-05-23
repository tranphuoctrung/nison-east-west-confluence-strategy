# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Pine Script v6 TradingView strategy** — a single `.pine` file deployed directly into TradingView Desktop. There is no build system, package manager, or test runner. The UI labels are in Vietnamese.

**Strategy name:** Nison East-West Confluence Strategy v6 - Quality Micro Breakout

## Working with the Strategy

**Editing:** Modify `Nison-East-West-Confluence-Strategy-v6-Quality-Micro-Breakout.pine` directly, then use the TradingView MCP tools to load and compile it.

**Loading into TradingView (via MCP):**
```
pine_set_source     → inject updated code into the open Pine editor
pine_smart_compile  → compile and surface any errors
pine_get_errors     → read compilation errors if compile fails
pine_get_console    → read runtime log output (plotted via log.* calls)
```

**Backtesting:** Use TradingView's built-in Strategy Tester — capture results with `data_get_strategy_results` or `capture_screenshot` with region `"strategy_tester"`.

**Reading live indicator output:**
```
chart_get_state          → current symbol, timeframe, indicator entity IDs
data_get_study_values    → current numeric values (RSI, MACD, etc.)
data_get_pine_tables     → dashboard table rows (win rate, confluence scores, etc.)
data_get_pine_labels     → on-chart annotations (entry/SL/TP levels)
```

## Architecture of the Pine Script

The file is organized into numbered sections (search for the section comment headers):

| Section | Purpose |
|---------|---------|
| `1. INPUT SETTINGS` | All user-configurable parameters (1.1–1.13). Grouped by module. |
| `2. CALCULATIONS` | HTF trend via `request.security()`, ATR, EMAs, RSI, MACD, Fibonacci swing detection, pivot-based S/R zones stored in arrays. |
| `3. PATTERN DETECTION` | Per-bar functions that return `[bool signal, float entryPrice, float stopLoss]`. Each pattern is a self-contained block. |
| `4. CONFLUENCE SCORING` | Aggregates RSI gate, MACD histogram direction, Fibonacci zone check, ADX strength into a score compared against `westernMinScore`. |
| `5. ENTRY LOGIC` | Combines pattern signal + confluence score + HTF trend filter + anti-chase/room filters → calls `strategy.entry()`. |
| `6. EXIT LOGIC` | Manages TP1/TP2 take-profit levels and trailing/fixed stop loss via `strategy.exit()`. |
| `7. VISUALS` | Dashboard table (`table.new`), entry/SL/TP lines, label tooltips. Arrays capped at 40 entries for display. |

## Key Design Patterns

**HTF data:** Fetched once with `request.security()` at bars_back=1 (previous confirmed bar) to avoid repainting. Two configurable HTF timeframes (default 60m and 240m), compared against fast/slow EMAs.

**S/R zones:** Stored in parallel arrays (`srPrices`, `srIsHigh`, `srAges`). Zones expire after `maxSRAgeBars` bars. New pivots are detected on right-bar confirmation (`ta.pivothigh/low`).

**Pattern modules (Section 3):** Each pattern (SingleCandle, ThreeBar, MorningStar, RejectionCluster, Continuation, MicroBreakout) is guarded by its own `useXxx` boolean input so it can be toggled independently.

**Confluence score:** Integer 0–3 counting how many of RSI/MACD/Fibonacci conditions pass. Entry requires score ≥ `westernMinScore` (default 1).

**Entry confirmation:** Controlled by `confirmedBarOnly` — when true, entries only fire on bar close (`process_orders_on_close=true` is already set at the strategy declaration).

**Risk management:** SL is pattern-structural (wick extreme + ATR buffer). TP1 and TP2 are SL-distance multiples (`rr1`, `rr2`). Position size defaults to 10% of equity.

## Input Parameter Groups

When adjusting parameters, locate the relevant input section:

- `1.1` — HTF timeframes and EMA lengths for trend filter
- `1.2` — Pivot lookback and S/R zone ATR width
- `1.3` — Reversal pattern toggles and per-pattern thresholds
- `1.4` — Continuation/pullback setup parameters
- `1.5` — Micro compression breakout parameters
- `1.6` — Entry confirmation candle requirements
- `1.7` — TP1/TP2 R:R ratios and SL ATR buffer
- `1.8–1.10` — RSI, MACD, Fibonacci confluence filter settings
- `1.11–1.13` — Quality score threshold, ADX, anti-chase, and visual display options
