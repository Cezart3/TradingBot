# TradingBot

An algorithmic **day-trading bot for US30 (Dow Jones CFD)**, built around an
**Opening Range Breakout (ORB)** strategy with a **machine-learning trade filter**,
running live against **MetaTrader 5**.

> ### 🔒 Why this repository is a description only
> The full source code is kept **private**. The bot is currently in a **demo
> forward-testing phase** — if it holds up in live conditions, the plan is to
> **monetize it** (paid access / managed signals). Publishing the strategy, the
> feature set and the trained model would give away the entire edge, so the code
> stays closed for now. This page documents what it does and how it's built.
>
> Happy to walk through the code or the validation in detail on request.

---

## What it does

At the New York equity open (9:30 ET, detected automatically in broker time) the
bot builds a **15-minute opening range** from M5 candles, then trades the breakout:

1. The first M5 close above the range triggers a **long**; below, a **short**.
2. Low-quality setups are filtered out by rule (range too small, weakest weekday,
   higher-timeframe RSI disagreeing with the direction).
3. Every surviving signal is scored by a **calibrated XGBoost / LightGBM ensemble**
   over **67 features** (range geometry, volume profile, and multi-timeframe
   indicators from M5 up to D1). Trades below a learned win-probability threshold
   are rejected.
4. Stop loss sits at the **volume-profile POC** of the range, take profit at **2R**;
   losing trades may re-enter once. Everything is flat by end of session.

**Risk management:** fixed 1% risk per trade, halved after two consecutive losses,
with a daily-loss breaker and a max-drawdown breaker (prop-firm-style limits).
State survives restarts.

## How it was validated

The model was trained on **~3,000 historical ORB trades (2016–2025)** with a
**time-ordered train / calibration / test split** and **walk-forward evaluation** —
each fold trains only on the past and is tested on unseen months. All walk-forward
folds came out positive in simulation (ECN spread modeled). A higher-win-rate 1:1
variant was trained the same way and **rejected** because its edge wasn't
statistically distinguishable from zero — the 2R configuration won.

A **feature-parity test** asserts that the live feature builder produces exactly
the same vectors as the training pipeline — the place this kind of system usually
breaks silently.

> ⚠️ Backtest and walk-forward results do **not** guarantee future performance.
> The bot has **never traded real money**; it is in a demo forward-test.

## Tech stack

- **Python** — core engine, strategy state machine, live runner
- **MetaTrader 5** API — live market data and execution (retry logic, spread
  buffer, position sizing)
- **XGBoost / LightGBM** + **scikit-learn** — the calibrated trade-filter ensemble
- **pandas / NumPy / PyArrow** — data pipeline and feature engineering
- **Optuna**, purged cross-validation, probability calibration, SHAP — model
  training (Colab notebook)
- **pydantic / pydantic-settings** — typed config and data models
- **CustomTkinter** — desktop dashboard (multi-account management, live equity /
  drawdown / breaker state, presets, logs)
- **pytest** — feature-parity and regression tests

## Architecture (high level)

```
brokers/      MetaTrader 5 wrapper (orders, sizing, spread handling)
strategies/   ORB state machine + live feature builder + model bundle loader
ml/           dataset builder; training in a Colab notebook (not shipped)
backtest/     backtest engine with SL/TP/filter grids
execution/    live runner (paper / demo / live modes)
utils/        risk manager, indicators, news filter, logging, time utils
gui/          CustomTkinter dashboard (multi-account, presets, live monitoring)
```

## Status

🟡 **Demo forward-testing.** Backtested and walk-forward simulated; live-validated
next. Monetization is considered only after the demo phase holds up.

---

### Contact

- 💼 [LinkedIn](https://www.linkedin.com/in/tocaciu-cezar-0865373b6/)
- 📧 cezartocaciu233@gmail.com

<sub>© 2026 Cezar Tocaciu. Code private. This page is documentation only.</sub>
