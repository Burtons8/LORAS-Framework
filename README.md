Python script that generates a clean “fact sheet” for a simple crypto momentum/rotation research strategy using Yahoo Finance price data and a risk-free proxy from FRED. Includes a walk-forward style backtest, allocation history, drawdowns, and a “live desk” table for research monitoring. For educational/research use only.

README.md (copy/paste)
# LORAS “Home Run” Strategy Fact Sheet (Research / Educational)

This repository contains a single Python script (Colab-friendly) that downloads historical crypto price data and produces a clean, white-background “Fact Sheet” style report showing:

- Strategy methodology (plain-English)
- Historical cumulative performance vs. BTC benchmark (log scale)
- Drawdown profile
- Historical allocation (stacked weights)
- A “Live Trade Desk” table that summarizes the most recent signals, suggested allocations, leverage state, and stop levels

IMPORTANT: This project is designed for **research and education**. It is **not** a trading system, does **not** place orders, and should not be used as the sole basis for any investment decision.

---

## Compliance & Legal Disclaimer (Read This First)

### Not investment advice
This code and its outputs are provided **for informational and educational purposes only**. Nothing here constitutes investment, legal, tax, or accounting advice, and nothing is a recommendation to buy, sell, or hold any security, digital asset, or investment product.

### No client relationship / no fiduciary duty
Use of this repository does not create an advisory relationship. No fiduciary duty is assumed or implied.

### No solicitation / no offer
This repository does not constitute an offer, solicitation, or marketing of advisory services, securities, or investment strategies.

### Risk warning
Digital assets are highly volatile and can experience rapid and material losses. Backtests can be misleading and do not reflect live trading conditions.

### Limitation of liability
To the maximum extent permitted by law, the author(s) and any affiliated firm are not liable for any direct or indirect losses arising from the use of this code, its outputs, or any reliance on it. Use at your own risk.

### “Research only” means you
You are responsible for validating assumptions, data quality, suitability, compliance requirements, and implementation details before using anything derived from this repository.

---

## What This Code Does (High-Level)

1. **Fetches daily price data** for BTC, ETH, and SOL from Yahoo Finance via `yfinance`
2. **Fetches a risk-free proxy** (3-month Treasury rate) from FRED via `pandas_datareader`
3. **Computes indicators** used by the strategy:
   - Trend filter: EMA(10) vs EMA(40)
   - Breakout filter: price vs prior 20-day high (Donchian-style)
   - Momentum score: (Price / SMA50) × (RSI / 50)
   - Stop level: ATR-based trailing stop (ATR(14) × multiplier)
4. **Runs a simple walk-forward backtest**:
   - Monthly rebalance cadence (approx. 21 trading days)
   - Weights set by positive momentum score proportions
   - “BTC Guard” regime rule that blocks ETH/SOL exposure when BTC is below its SMA50 (weight goes to cash)
   - Dynamic leverage: 1x in trend, 2x if breakout condition is also met
   - Adds a small daily “drag proxy” as a simplified cost estimate
   - Unallocated capital earns the risk-free daily rate
5. **Generates a single-page Fact Sheet visualization** with performance, risk metrics, allocation, and a live desk table.

---

## Strategy Logic (As Implemented)

### Universe
- BTC-USD, ETH-USD, SOL-USD

### Rebalance frequency
- Monthly (every ~21 trading days)

### Allocation (Rotation)
On each rebalance date:
- Compute a **Momentum Score** for each asset:
  - `MomentumScore = (Price / SMA50) * (RSI / 50)`
- Negative or zero scores are floored at 0
- Allocate weights in proportion to positive scores:
  - `w_i = score_i / sum(scores)`

### Regime filter (“BTC Guard”)
If BTC is below its SMA50 on rebalance date:
- ETH and SOL weights are set to 0
- The blocked allocation becomes cash (i.e., uninvested)

### Trend and Breakout Signals (Execution)
Daily execution uses the previous day’s signal values (to reduce lookahead):
- Trend signal: EMA(10) > EMA(40)
- Breakout signal: today’s price > **yesterday’s** rolling 20-day high

Leverage:
- If not trending → 0x exposure (cash)
- If trending but not breakout → 1x exposure
- If trending and breakout → up to 2x exposure (configurable)

### Stop Loss (Displayed)
The script computes a stop level:
- `Stop = Price - ATR(14) * ATR_MULTIPLIER`

Note: In the current script, the stop is **calculated and displayed** in the Fact Sheet and “Live Desk,” but the backtest does **not** explicitly simulate intraday stop execution (see Limitations).

---

## Outputs

### Fact Sheet sections
1. Header + strategy methodology box
2. Cumulative performance chart (log scale) vs BTC benchmark
3. Drawdown chart
4. Allocation history stack plot
5. “Live Trade Desk” table:
   - Latest price
   - Suggested allocation percentage
   - Leverage state
   - Stop price (ATR-based)
   - Status labels (e.g., BREAKOUT, TRENDING, BLOCKED)

### Backtest metrics (shown on the chart)
- CAGR (simplified annualization from mean daily returns)
- Volatility (annualized)
- Sharpe (simplified; see limitations)
- Max drawdown

---

## Installation

### Option A: Google Colab (recommended)
Just paste the script into a Colab cell and run. The script attempts to install:
- `yfinance`
- `pandas_datareader`

### Option B: Local Python
Recommended Python version: 3.9+

```bash
pip install yfinance pandas_datareader pandas numpy matplotlib statsmodels

How To Run
python loras_fact_sheet.py


Or in Colab, run the block as-is. It will:

download data

run the backtest

display the Fact Sheet figure

Configuration

Edit the Config class:

TICKERS: assets included

START_DATE: backtest start date

MAX_LEVERAGE: leverage cap (e.g., 2.0)

ATR_PERIOD, ATR_MULTIPLIER: stop calculation

TRAIN_WINDOW_YEARS: warm-up window before simulation starts

TEST_WINDOW_MONTHS: rebalance frequency (currently monthly)

Known Limitations (Important)

This repository intentionally keeps the model simple for research illustration. As a result:

Data source limitations

Yahoo Finance crypto data may have missing values, timing quirks, exchange differences, and revisions.

“Adj Close” behavior for crypto is not the same as for equities.

Lookahead and execution realism

The script attempts to reduce lookahead by using prior-day signals for daily decisions.

However, it does not model realistic execution (bid/ask spreads, slippage, market impact, funding rates).

Transaction costs

Uses a simplified daily drag proxy (abs(pos) * 0.0001) which is not a true cost model.

Stops not simulated

ATR stop levels are calculated and displayed, but the backtest does not simulate intraday stop hits or stop order fills.

Risk-free proxy

Uses 3-month Treasury rate from FRED, resampled and converted to a daily approximation. This is a convenience proxy.

Sharpe ratio simplification

Sharpe is computed as annualized mean divided by annualized volatility, without fully integrating excess returns vs. risk-free at each step.

Leverage realism

“Leverage” here is a numerical multiplier used in the math. It does not model margin requirements, liquidation risk, borrow costs, or exchange-specific leverage mechanics.

Intended Use

Use this project to:

Learn how to build a repeatable research report (fact sheet)

Explore simple momentum/rotation concepts

Visualize performance, drawdowns, and allocations

Prototype additional realism (fees, slippage, stop simulation, walk-forward validation)

Not intended for:

Live trading

Client reporting

Marketing materials

Performance advertising

Investment recommendations

Ethics & Attribution

If you publish results derived from this code, please:

disclose assumptions and limitations

avoid presenting backtests as real performance

provide full methodology and dates

cite data sources (Yahoo Finance, FRED)

License

Suggested: MIT License (see LICENSE).
