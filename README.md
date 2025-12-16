# Dual Momentum Strategy Inspired Fund
## Quantitative Finance Final Project - Insper São Paulo

**Course:** Quantitative Finance  
**Institution:** Insper  
**Period:** January 2015 - December 2024  
**Universe:** 40 stocks from S&P 500  
**Team:** Anna-Luisa Horst, Jack Heyworth, Lorenzo Martinez, Noelia Barreira

---

## Executive Summary

This project presents a comprehensive systematic equity investment strategy based on **dual momentum principles**. The strategy combines cross-sectional momentum (relative strength) with time-series momentum (absolute trend) to identify optimal entry and exit points in equity markets.

Our analysis demonstrates:
- **Long-Only Dual Momentum**: 21.2% annualized return, 1.04 Sharpe ratio
- **Outperformance**: 7.7% alpha vs. equal-weight benchmark (13.5% return)
- **Risk Management**: Volatility targeting reduces volatility from 20.3% to 14.7%

---

## Investment Strategy

### Core Concept

Dual Momentum combines two complementary signals:

1. **Cross-Sectional Momentum**
   - Rank all stocks by recent performance (3-month lookback)
   - Long the top 15% performers (winners)
   - Short the bottom 15% performers (losers)

2. **Time-Series Momentum** 
   - Filter cross-sectional signals using absolute trends
   - Only go long if stock has positive 3-month return
   - Only go short if stock has negative 3-month return
   - Avoids "catching falling knives" and "shorting rising stars"

### Two Implementations

#### 1. Market-Neutral Long-Short (LS)
- **Concept**: Equal dollar exposure to long and short positions
- **Target**: Zero net market exposure, pure alpha generation
- **Result**: Underperformed due to bull market environment (2015-2024)
- **Sharpe Ratio**: 0.13 (raw), 0.15 (vol-targeted)

#### 2. Long-Only Dual Momentum
- **Concept**: Concentrated exposure to strongest momentum signals on long side only
- **Target**: Capture directional market trends with reduced drawdowns
- **Result**: Strong outperformance vs. benchmark
- **Sharpe Ratio**: 1.04 (raw), 0.81 (vol-targeted)

---

## Universe Construction

### Selection Criteria
- **Base Universe**: S&P 500 constituents as of January 2015
- **Final Universe**: 40 stocks with sector-balanced representation
- **Sectors**: ~4 representative stocks per sector (10 sectors)
- **Liquidity**: All are large-cap, highly liquid securities
- **Time Period**: January 2015 - December 2024 (10-year backtest)

### Universe Composition

| Sector | Stocks | Examples |
|--------|--------|----------|
| Technology | 4 | AAPL, GOOGL, MSFT, META |
| Energy | 4 | XOM, CVX, COP, SLB |
| Financials | 4 | BRK-B, WFC, JPM, BAC |
| Healthcare | 3 | JNJ, PFE, MRK, UNH |
| Industrials | 4 | GE, BA, UPS, CAT |
| Consumer | 4 | AMZN, HD, DIS, MCD, WMT, PG, KO, PEP |
| Materials | 2 | NEM, FCX |
| Utilities | 3 | NEE, D, SO, EXC |
| Telecom | 3 | VZ, T, CMCSA |
| Other | 2 | CHTR, DD, DOW |

---

## Methodology

### Signal Generation

#### Step 1: Calculate Returns
```python
# 3-month lookback horizon (t-3 to t)
returns_lookback = df_monthly.pct_change(LOOKBACK_MONTHS)
```

#### Step 2: Cross-Sectional Ranking
```python
# Rank all stocks by 3-month performance (1 = best, N = worst)
ranks = returns_lookback.rank(axis=1, ascending=False)

# Select top 15% (winners) and bottom 15% (losers)
long_signals = ranks <= cutoff_percentile
short_signals = ranks >= (N - cutoff_percentile)
```

#### Step 3: Time-Series Filter
```python
# Only take positions that align with absolute trend
# Long: only if 3-month return > 0
# Short: only if 3-month return < 0

long_candidates = long_signals & (returns_lookback > 0)
short_candidates = short_signals & (returns_lookback < 0)
```

#### Step 4: Portfolio Construction
```python
# Equal-weight within long and short baskets
long_weights = long_candidates / long_candidates.sum(axis=1)
short_weights = short_candidates / short_candidates.sum(axis=1)

# Calculate forward 1-month returns
portfolio_return = (long_weights * returns_forward).sum(axis=1) - \
                  (short_weights * returns_forward).sum(axis=1)
```

### Risk Management: Volatility Targeting

To manage portfolio risk dynamically and prevent extreme drawdowns:

```python
# Measure realized volatility (6-month rolling std)
realized_vol = returns.rolling(VOLLOOKBACK).std() * np.sqrt(12)

# Calculate leverage to maintain target volatility
leverage_scalar = TARGET_VOL / realized_vol
leverage_scalar = leverage_scalar.clip(upper=MAX_LEVERAGE)  # Cap at 1.5x

# Scale portfolio weights
managed_returns = raw_returns * leverage_scalar
```

**Effects:**
- **Low volatility markets**: Scale up to 1.5x leverage to maintain consistent returns
- **High volatility markets**: Reduce exposure proportionally, limiting drawdowns

---

## Performance Results (2015-2024)

### Raw Returns (Unlevered)

| Strategy | Annual Return | Volatility | Sharpe Ratio | Max Drawdown |
|----------|--------------|-----------|------------|---------------|
| LS Return | 2.6% | 19.9% | 0.13 | -45.2% |
| Long-Only Return | 21.2% | 20.3% | 1.04 | -38.7% |
| Equal-Weight Benchmark | 13.5% | 15.3% | 0.88 | -34.5% |
| S&P 500 (Conceptual) | ~13-14% | ~16-17% | ~0.80-0.85 | ~-30% to -35% |

### Volatility-Targeted Results

| Strategy | Annual Return | Volatility | Sharpe Ratio |
|----------|--------------|-----------|------------|  
| LS Vol-Targeted | 12.0% | 15.1% | 0.15 |
| Long-Only Vol-Targeted | 14.7% | 12.0% | 0.81 |
| Equal-Weight Vol-Targeted | ~13% | ~12% | ~0.88 |

### Key Observations

1. **Long-Only Outperformance**
   - 7.7% alpha vs. equal-weight benchmark
   - 2.1x Sharpe ratio improvement (1.04 vs. 0.88)
   - Sustained outperformance throughout decade

2. **Market-Neutral Underperformance**
   - Structural bull market environment (2015-2024)
   - Short book systematically underperformed
   - Limited universe breadth (40 stocks)
   - High within-sector correlation offset long/short gains

3. **Volatility Targeting Trade-offs**
   - Reduces return in strong uptrend periods
   - Protects capital during drawdowns
   - Improves Sharpe ratio for long-only strategy (0.81)

---

## Key Findings

### What Worked
✅ **Momentum Premium in Trending Markets**: Long-biased momentum captured persistent outperformance in bull market
✅ **Time-Series Filter Value**: Avoiding counter-trend positions improved risk-adjusted returns
✅ **Volatility Targeting Benefits**: Reduced peak drawdowns and volatility spikes

### What Didn't Work
❌ **Market-Neutral in Bull Market**: Short book faced structural headwinds
❌ **Limited Universe Breadth**: Only 40 stocks insufficient for robust market-neutral strategies
❌ **Sector Clustering**: Sector-balanced selection created high within-sector correlation, offsetting diversification benefits

### Critical Lessons

1. **Universe Construction Matters**
   - Larger universes (500+ stocks) needed for market-neutral
   - Strict liquidity filters (volume, bid-ask spreads) essential
   - Correlation structure drives portfolio construction

2. **Signal Quality**
   - Simple 3-month momentum insufficient for all regimes
   - Multiple timeframe signals improve robustness
   - Trend confirmation filters reduce false signals

3. **Market Regime Dependency**
   - Momentum strategies excel in trending markets
   - Sideways/mean-reverting markets favor different signals
   - Regime detection can improve strategy adaptability

---

## Project Structure

```
.
├── README.md                          # This file
├── METHODOLOGY.md                     # Detailed technical explanation
├── requirements.txt                   # Python dependencies
├── .gitignore                         # Git ignore rules
│
├── src/
│   ├── __init__.py
│   ├── strategy.py                    # Core dual momentum logic
│   ├── data_loader.py                 # Data loading and preprocessing
│   ├── risk_management.py             # Volatility targeting implementation
│   ├── metrics.py                     # Performance calculation and analysis
│   └── visualization.py               # Plotting and analysis visualization
│
├── data/
│   ├── raw/
│   │   └── SPXdatabase2010.xlsx       # Historical price data
│   ├── processed/
│   │   └── returns_by_stock.csv       # Processed returns
│   └── results/
│       └── backtest_results.json      # Strategy performance output
│
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_strategy_backtesting.ipynb
│   ├── 03_performance_analysis.ipynb
│   └── 04_risk_analysis.ipynb
│
└── results/
    ├── cumulative_returns.png         # Performance plot
    ├── correlation_heatmap.png        # Universe correlation structure
    └── performance_metrics.csv        # Detailed metrics table
```

---

## Installation & Usage

### Requirements
- Python 3.8+
- pandas, numpy, matplotlib, scipy
- See `requirements.txt` for full list

### Setup

```bash
# Clone repository
git clone https://github.com/lorenlorenloren/Final-Quant-Finance-Project---Insper-Sao-Paulo.git
cd Final-Quant-Finance-Project---Insper-Sao-Paulo

# Install dependencies
pip install -r requirements.txt

# Run backtest
python src/strategy.py
```

### Main Script

```python
from src.strategy import run_dual_momentum
from src.data_loader import load_data

# Load data
data = load_data('data/raw/SPXdatabase2010.xlsx', 
                 tickers=TARGET_TICKERS)

# Run strategy
results = run_dual_momentum(data)

# Analyze results
stats = calculate_metrics(results)
print(stats)
```

---

## Recommendations for Future Work

### 1. Expand Universe Size
- Increase to 100-200 stocks with stricter liquidity filters
- Improves diversification and reduces concentration risk
- Enables more robust market-neutral implementation

### 2. Enhance Signal Quality
- Replace simple 3-month returns with multi-timeframe signals
- Implement 200-day moving average crossover filter
- Add relative strength index (RSI) for oversold/overbought detection

### 3. Improve Risk Management
- Implement position-level stop losses
- Add maximum drawdown triggers
- Use dynamic leverage based on volatility regime

### 4. Incorporate Transaction Costs
- Model realistic slippage and bid-ask spreads
- Estimate portfolio turnover and commissions
- Stress-test strategy net of frictions

### 5. Develop Regime Detection
- Identify bull/bear/sideways markets
- Adapt signals to market conditions
- Reduce drawdowns in mean-reverting regimes

---

## References

### Academic & Practitioner Sources
- Antonacci, G. (2014). "Dual Momentum Investing" - *McGraw Hill*
- Carhart, M. (1997). "Mutual Fund Performance" - *Journal of Finance*
- Jegadeesh, N. & Titman, S. (1993). "Returns to Buying Winners and Selling Losers" - *Journal of Finance*

### Data Source
- S&P 500 historical prices (2010-2024)
- Yahoo Finance / Bloomberg terminal

---

## Disclaimer

This is an academic project for educational purposes only. All results are based on historical backtesting and should not be considered as investment advice. Past performance does not guarantee future results. Actual trading involves risks including potential loss of principal.

---

## Author
Lorenlorenloren - Quantitative Finance Student, Insper São Paulo  
**Team Members**: Anna-Luisa Horst, Jack Heyworth, Lorenzo Martinez, Noelia Barreira

**Last Updated**: December 2024  
**Status**: Final Project Submission
