# Option Premium Scalping Assistant Architecture (TSE)

## Goal and scope
- Focus only on **Long Call** and **Long Put** positions.
- No short options and no hold-to-expiry behavior.
- Objective: buy undervalued premium and exit at higher premium with tight risk controls.

## System modules

### 1) Data layer (`src/data`)
Collects and normalizes historical + live streams:
- Underlying stock ticks/candles.
- Option chain (bid/ask/last, OI, volume, depth).
- Market-wide indicators:
  - Index trend in real time.
  - Buy/sell queue pressure.
  - Retail vs institutional flow.
  - Code-to-code transfer indicators.
  - Suspicious/fake money flow proxies.

Outputs:
- Unified feature-ready tables (time-aligned).
- Quality flags (missing ticks, stale quotes, abnormal spread).

### 2) Feature engine (`src/features`)
Builds model and rule inputs from market microstructure:
- Price/return momentum windows.
- Volatility regime (ATR, realized vol).
- Option microstructure features:
  - Spread%, depth imbalance, quote refresh rate.
  - Premium velocity and acceleration.
- Flow and participation features:
  - Volume anomaly score.
  - Retail/institutional net flow.
  - Code-to-code pressure score.
  - Fake money flow score.
- Index-relative features:
  - Symbol beta-adjusted move vs index.
  - Divergence with index trend.

### 3) Fair-value and Greeks engine (`src/models/fair_value.py`)
- Calculates theoretical premium (BS-like baseline + local adjustments).
- Computes Greeks (Delta, Gamma, Theta, Vega).
- Produces mispricing scores:
  - `undervaluation_score`
  - `theta_penalty`
  - `liquidity_penalty`

### 4) Signal engine (`src/signals`)
Two-stage gating:
1. **Underlying direction gate** (short-horizon directional probability).
2. **Premium expansion gate** (probability premium can grow short-term).

Entry only if:
- Mispricing is positive and significant.
- Liquidity + spread constraints pass.
- Flow/participation filters support move.
- Index trend is not strongly contradictory.

### 5) Risk engine (`src/risk`)
- Per-trade max risk budget.
- Dynamic stop-loss and take-profit.
- Max holding time.
- Daily drawdown kill-switch.
- Exposure caps by underlying and sector.

### 6) Execution assistant (`src/execution`)
- Smart order placement around bid/ask.
- Requote logic when spread shifts.
- Partial fill handling.
- Exit prioritization when liquidity degrades.

### 7) Backtesting and simulation (`src/backtest`)
- Event-driven simulation with bid/ask realism.
- Slippage + fee + delay models.
- Walk-forward validation.
- Regime-wise performance breakdown.

### 8) Monitoring and journaling (`src/monitoring`)
- Trade journal with full reason codes.
- Live health checks for data and model drift.
- Daily post-trade analytics dashboard.

## Repository structure

```text
predictor/
  config/
    universe.yml
    risk.yml
    feature_flags.yml
  docs/
    architecture.md
  notebooks/
  src/
    data/
      ingestion.py
      normalization.py
      quality.py
    features/
      price_features.py
      option_features.py
      flow_features.py
      market_regime_features.py
    models/
      fair_value.py
      direction_model.py
      premium_model.py
      contract_ranker.py
    signals/
      entry_rules.py
      exit_rules.py
      signal_pipeline.py
    risk/
      position_sizing.py
      limits.py
      kill_switch.py
    execution/
      order_router.py
      fill_manager.py
    backtest/
      engine.py
      costs.py
      walk_forward.py
    monitoring/
      journal.py
      diagnostics.py
    common/
      types.py
      constants.py
      utils.py
  tests/
    test_signal_pipeline.py
    test_risk_limits.py
```

## Key feature families to include from day one
1. Liquidity: spread%, depth, turnover.
2. Flow: real/legal net flow, buy-power, volume shocks.
3. Code-to-code behavior signals.
4. Fake money flow proxies.
5. Index trend and market breadth in real time.
6. Time-to-expiry and theta pressure.
7. Mispricing vs theoretical value.

## MVP build order
1. Data + quality checks.
2. Rule-based fair-value + liquidity filter.
3. Basic entry/exit + risk limits.
4. Backtest with bid/ask realism.
5. Add ML models for direction and premium expansion.
