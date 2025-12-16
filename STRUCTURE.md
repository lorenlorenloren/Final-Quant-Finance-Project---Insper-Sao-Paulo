# Project Structure

## Dual Momentum Strategy - Quantitative Finance Final Project

```
Final-Quant-Finance-Project---Insper-Sao-Paulo/
│
├── README.md                           # Main documentation & executive summary
├── STRUCTURE.md                        # This file - project structure guide
├── requirements.txt                    # Python dependencies
├── .gitignore                          # Git ignore rules
│
├── src/                                # Source code for the strategy
│   ├── __init__.py
│   ├── strategy.py                     # Core dual momentum logic & backtesting
│   ├── data_loader.py                  # Load Excel data & preprocessing
│   ├── risk_management.py              # Volatility targeting implementation
│   ├── metrics.py                      # Performance metrics & Sharpe ratio
│   └── visualization.py                # Plotting cumulative returns & heatmaps
│
├── data/                               # Data files
│   ├── raw/
│   │   └── SPXdatabase2010.xlsx        # Historical S&P 500 prices (input)
│   ├── processed/
│   │   └── returns_by_stock.csv        # Calculated returns (intermediate)
│   └── results/
│       └── backtest_results.json       # Strategy performance output
│
├── notebooks/                          # Jupyter notebooks for analysis
│   ├── 01_data_exploration.ipynb       # EDA & data quality checks
│   ├── 02_strategy_backtesting.ipynb   # Run full backtest
│   ├── 03_performance_analysis.ipynb   # Metrics & visualizations
│   └── 04_risk_analysis.ipynb          # Drawdown & volatility analysis
│
├── results/                            # Output files
│   ├── cumulative_returns.png          # Performance comparison chart
│   ├── correlation_heatmap.png         # Universe correlation matrix
│   ├── performance_metrics.csv         # Annualized returns, Sharpe, etc.
│   └── backtest_summary.json           # Final performance report
│
└── docs/                               # Additional documentation
    └── methodology.md                  # Detailed technical explanation
```

## Key Files Explained

### Core Implementation

- **`src/strategy.py`**: Main backtesting engine
  - `run_dual_momentum()` - Execute full strategy
  - Cross-sectional ranking logic
  - Time-series filtering
  - Portfolio weight calculations

- **`src/data_loader.py`**: Data handling
  - `load_data()` - Read Excel prices
  - `resample_to_monthly()` - Convert daily to monthly
  - Handle missing data / NaNs

- **`src/risk_management.py`**: Volatility targeting
  - `apply_vol_targeting()` - Dynamic leverage scaling
  - 6-month rolling volatility calculation
  - 1.5x maximum leverage cap

- **`src/metrics.py`**: Performance analysis
  - `calculate_metrics()` - Sharpe ratio, max drawdown, etc.
  - `calculate_annual_returns()` - Annualize returns
  - `calculate_win_rate()` - Percentage of positive months

### Configuration

Strategy parameters in `src/strategy.py`:
```python
LOOKBACK_MONTHS = 3           # 3-month momentum window
PERCENTILE_CUTOFF = 0.15      # Top/bottom 15% of stocks
START_YEAR = 2015             # Backtest start date
END_YEAR = 2024               # Backtest end date
TARGET_TICKERS = [...]        # 40-stock universe
TARGET_VOL = 0.12             # 12% target volatility
VOL_LOOKBACK = 6              # 6-month rolling window for vol
MAX_LEVERAGE = 1.5            # Maximum leverage cap
```

## Data Flow

```
SPXdatabase2010.xlsx
    ↓
data_loader.load_data()
    ↓
Monthly Returns (DataFrame)
    ↓
strategy.run_dual_momentum()
    │
    ├── Cross-sectional ranking
    ├── Time-series filtering
    ├── Portfolio construction
    └── Risk management
    ↓
Strategy Returns (Series)
    ↓
metrics.calculate_metrics()
    ↓
Performance Report (JSON/CSV)
```

## Performance Outputs

The backtest generates:

1. **backtest_results.json**
   - Daily/monthly returns for each strategy
   - Cumulative performance curves
   - Strategy metadata (dates, universe size, etc.)

2. **performance_metrics.csv**
   ```
   Strategy,Annual Return,Volatility,Sharpe,Max Drawdown
   LS Raw,2.6%,19.9%,0.13,-45.2%
   Long-Only Raw,21.2%,20.3%,1.04,-38.7%
   LS Vol-Targeted,12.0%,15.1%,0.15,-28.5%
   Long-Only Vol-Targeted,14.7%,12.0%,0.81,-22.3%
   Equal-Weight Bench,13.5%,15.3%,0.88,-34.5%
   ```

3. **Charts**
   - Cumulative returns plot (PNG)
   - Correlation heatmap (PNG)
   - Drawdown analysis (PNG, if generated)

## Dependencies

See `requirements.txt` for complete list:
- **pandas**: Data manipulation
- **numpy**: Numerical calculations
- **matplotlib/seaborn**: Visualization
- **scipy**: Statistical functions
- **openpyxl**: Excel reading

## Usage

### Run Full Backtest
```bash
python src/strategy.py
```

### Interactive Analysis
```bash
jupyter notebook notebooks/02_strategy_backtesting.ipynb
```

### Quick Test
```python
from src.strategy import run_dual_momentum
from src.data_loader import load_data

data = load_data('data/raw/SPXdatabase2010.xlsx')
results = run_dual_momentum(data)
print(results.describe())
```

## Future Enhancements

- [ ] Add multi-timeframe signals (1M, 3M, 12M lookbacks)
- [ ] Implement stop-loss mechanism
- [ ] Model transaction costs
- [ ] Sector rotation overlay
- [ ] Machine learning signal generation
- [ ] Regime detection for signal adaptation

---

**Last Updated**: December 2024  
**Team**: Anna-Luisa Horst, Jack Heyworth, Lorenzo Martinez, Noelia Barreira
