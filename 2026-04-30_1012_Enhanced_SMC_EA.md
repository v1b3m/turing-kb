# Enhanced SMC EA

`ea-enhanced.mq5` is the primary Expert Advisor in this repository. It is now profile-driven, with the default direction aimed at `XAUUSD` first instead of forcing one set of assumptions across gold, forex, oil, and crypto.

## What It Is

This EA combines:

- higher-timeframe bias using RSI and EMA alignment
- entry-timeframe confirmation using RSI, MFI, EMA trend, Fair Value Gaps, Order Blocks, and structure breaks
- profile-based session and spread handling
- ATR-based stop placement and risk-based lot sizing
- ongoing trade management through break-even, trailing stop, and partial close logic
- end-of-run diagnostics that show why entries were filtered or rejected

## What It Is Optimized For

- XAU-first intraday testing, with `M15` entry and `H1` higher-timeframe bias as the default shape
- Multi-timeframe discretionary-style logic expressed as rules
- Markets where spread behavior should be normalized against volatility instead of fixed points
- Traders who want drawdown limits, position caps, and active post-entry management
- Backtesting and iterative parameter tuning with clearer rejection diagnostics

## Core Strengths

- XAU-first preset with looser scoring than the previous shared-default version
- Profile presets for `XAU`, `FOREX_MAJOR`, `OIL`, `CRYPTO`, and `CUSTOM`
- Spread filtering based on spread relative to ATR for profile-driven modes
- Narrower location checks around daily levels instead of the old broad pivot-range confirmation
- Entry-risk and entry-ATR-based trade management so exits do not drift with later volatility
- Tighter short-side profit protection for the XAU preset

## Main Inputs To Tune

- `UseProfileOverrides`
- `AssetProfile`
- `RiskPercent`
- `MaxDailyDrawdownPct`
- `MaxOpenTrades`
- `RiskRewardRatio`
- `StopLossATRMult`
- `TrailingATRMult`
- `BreakEvenATRMult`
- `PartialClosePct`
- `PartialTPRatio`
- `EntryTF`
- `HTF`
- `TradingSession`
- `EntryMode`
- `MaxSpreadPoints`
- `EnableReasonCounters`

## Profiles

- `PROFILE_XAU`
  Gold-first preset. Uses London + New York session coverage, ATR-normalized spread filtering, looser momentum thresholds, and tighter short-side profit protection.
- `PROFILE_FOREX_MAJOR`
  Stricter than XAU. Intended for majors like `EURUSD` and `USDJPY`.
- `PROFILE_OIL`
  Experimental preset with unrestricted session coverage and wider ATR-based spread tolerance.
- `PROFILE_CRYPTO`
  Experimental preset with unrestricted session coverage and the widest ATR-based spread tolerance.
- `PROFILE_CUSTOM`
  Leaves manual inputs in control and keeps `TradingSession` / `MaxSpreadPoints` relevant.

When `UseProfileOverrides = true`, the selected profile controls the effective session, spread rule, confirmation requirement, and exit-management behavior.

## Best Fit

Use this EA when you want the most complete implementation in the repo and plan to optimize around:

- XAU behavior first
- symbol-specific session behavior
- volatility-normalized spread tolerance
- ATR-based stop sizing and exit behavior
- confirmation strictness and structure-location alignment

## Diagnostics

When `EnableReasonCounters = true`, the EA prints a tester summary on deinit showing:

- blocks from session, spread, drawdown, and max-open-trade filters
- long and short evaluation counts
- low-score rejection counts
- trend and structure hit counts
- level-proximity hits
- long and short entries taken

## Notes

- The SMC detection is heuristic, not institutional-grade market structure modeling.
- `PROFILE_XAU` is the main tuning path right now.
- `PROFILE_OIL` and `PROFILE_CRYPTO` should still be treated as experimental.
- Forward testing on demo is still necessary before any live use.
