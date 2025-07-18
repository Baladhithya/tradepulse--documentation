# TradePulse User Guide

## Getting Started

### Installation and Setup

#### Prerequisites

```
# Required software
- C++17 compiler (GCC 7+ or Clang 5+)
- CMake 3.10 or higher
- Python 3.7+ with pandas, matplotlib, seaborn
- Git for version control

# Optional but recommended
- Visual Studio Code with C++ extensions
- Python virtual environment
```

#### Build Process

```
# 1. Clone the repository
git clone <repository-url>
cd tradepulse

# 2. Create build directory
mkdir build && cd build

# 3. Configure and build
cmake ..
make -j4

# 4. Verify installation
./tradepulse --help
```

#### Data Setup

```
# Create required directories
mkdir -p data logs reports plots live_data

# Download sample data (optional)
python3 scripts/fetch_binance_ohlcv.py
```

## System Usage Tutorial

### 1. Running Your First Backtest

#### Basic Backtest Execution

```
# Start the application
./tradepulse

# Select mode: 1 (Backtest with Full Analytics)
# Choose assets: 1,2,3 (BTC, ETH, SOL)
# Select strategies: 1,2 (EMA-RSI, Momentum)
```

**Expected Output:**

```
 TradePulse - COMPREHENSIVE Trading System v2.0
================================================
Select mode:
1. Backtest with Full Analytics
2. Live Shadow with Risk Management
3. Strategy Optimization & Walk-Forward
4. Risk Management Demo
Enter choice: 1

Available assets:
1. BTCUSDT
2. ETHUSDT
3. SOLUSDT
Select assets (comma-separated indices): 1,2

Available strategies:
1. EMARSI
2. Momentum
3. SMA
4. Volatility
5. Breakout
Select strategies (comma-separated indices): 1,2
```

#### Understanding Results

```
 [EMARSI - BTCUSDT] Results:
   - Total Trades: 45
   - Win Rate: 62.2%
   - Cumulative Return: 15.7%
   - Sharpe Ratio: 1.23
   - Max Drawdown: 8.4%
   - Avg Slippage: 2.1 bps
```

### 2. Live Shadow Trading

#### Setting Up Live Data Streams

```
# Terminal 1: Start BTC data stream
python3 scripts/binance_stream.py 1

# Terminal 2: Start ETH data stream  
python3 scripts/binance_stream.py 2

# Terminal 3: Run live shadow system
./tradepulse
# Select mode: 2 (Live Shadow with Risk Management)
```

#### Monitoring Live Performance

```
 BTCUSDT @ 2024-01-15 10:30:00 | Price: $42,150 | Vol: 1.25 | Regime: 🐂 BULL
 BUY APPROVED: EMARSI | BTCUSDT | $42,150 | Size: 8% 
💼 Portfolio: $108,450.00 | P&L: $8,450.00 | Positions: 2
```

### 3. Strategy Development

#### Creating a New Strategy

1. **Define Strategy Class:**

```cpp
// include/strategies/strategy_custom.hpp
#pragma once
#include "strategy.hpp"

class CustomStrategy : public Strategy {
public:
    void on_data(const Candle& candle, const Candle* high_tf = nullptr) override;
    bool should_buy() const override;
    bool should_sell() const override;
    void enable_debug(const std::string& strategy_name, const std::string& symbol) override;

private:
    // Strategy-specific variables
    bool buy_signal = false;
    bool sell_signal = false;
    // Add your indicators and parameters
};
```

2. **Implement Strategy Logic:**

```cpp
// src/strategies/strategy_custom.cpp
#include "strategies/strategy_custom.hpp"

void CustomStrategy::on_data(const Candle& candle, const Candle* high_tf) {
    // Reset signals
    buy_signal = sell_signal = false;
    
    // Your strategy logic here
    if (/* your buy condition */) {
        buy_signal = true;
    }
    
    if (/* your sell condition */) {
        sell_signal = true;
    }
}

bool CustomStrategy::should_buy() const { return buy_signal; }
bool CustomStrategy::should_sell() const { return sell_signal; }
```

3. **Register Strategy:**

```cpp
// In main.cpp, add to strategy list:
{"Custom", []() { return std::make_unique<CustomStrategy>(); }}
```

#### Strategy Parameters

**Common Parameters to Consider:**

* **Lookback Periods**: How many candles to analyze
* **Thresholds**: Signal generation thresholds
* **Risk Controls**: Stop loss, take profit levels
* **Filters**: Volume, volatility, trend filters

**Example Parameter Configuration:**

```cpp
class MomentumStrategy : public Strategy {
private:
    int momentum_period = 10;        // Lookback period
    double threshold_pct = 0.5;      // Signal threshold
    double volume_multiplier = 1.2;  // Volume filter
    int cooldown_candles = 5;        // Prevent overtrading
};
```

### 4. Performance Analysis

#### Generated Reports

**1. Trade Logs (`logs/` directory):**

* `trades_STRATEGY_SYMBOL.csv`: Individual trade records
* `STRATEGY_SYMBOL_equity.csv`: Portfolio value over time
* `debug_STRATEGY_SYMBOL.csv`: Detailed strategy signals

**2. Performance Charts (`plots/` directory):**

* `STRATEGY_equity_curve.png`: Portfolio growth over time
* `STRATEGY_drawdown.png`: Drawdown analysis
* `STRATEGY_win_loss_distribution.png`: Trade outcome distribution
* `STRATEGY_pnl_histogram.png`: Profit/loss distribution

**3. HTML Reports (`reports/` directory):**

* Comprehensive performance analysis
* Risk metrics and statistics
* Interactive charts and tables

#### Interpreting Results

**Key Metrics to Monitor:**

1. **Sharpe Ratio**: Risk-adjusted returns
2. **Maximum Drawdown**: Largest peak-to-trough decline
3. **Win Rate**: Percentage of profitable trades
4. **Profit Factor**: Gross profit / Gross loss

### 5. Strategy Optimization

#### Parameter Optimization

```
# Run optimization mode
./tradepulse
# Select mode: 3 (Strategy Optimization & Walk-Forward)
```

The system will:

1. Test multiple parameter combinations
2. Perform walk-forward analysis
3. Generate stability and robustness scores
4. Save results to `reports/optimization_results.txt`

#### Walk-Forward Analysis

**Process:**

* Optimize parameters on historical data
* Test optimized parameters on unseen data
* Repeat with rolling windows
* Compare in-sample vs out-of-sample performance

### 6. Risk Management

#### Risk Limits Configuration

```cpp
RiskLimits limits;
limits.max_position_size = 0.1;     // 10% max position
limits.max_daily_loss = 0.02;       // 2% daily loss limit  
limits.max_drawdown = 0.05;         // 5% max drawdown
limits.max_positions = 5;           // Max concurrent positions
```

#### Real-Time Monitoring

```text
🚨 Risk Alerts:
⚠️ Approaching maximum drawdown limit
⚠️ High correlation detected between BTCUSDT and ETHUSDT
✅ All other risk metrics within limits
```

## Troubleshooting and FAQ

### Common Issues

* Build or dependency errors → Check compiler/CMake versions
* Data not loading → Ensure CSVs exist in `data/`
* Performance slow → Use smaller data for debugging
* Live stream not working → Check network, Python dependencies

### Frequently Asked Questions

* How to add a strategy → Implement class, register in main
* How to use custom data → Modify `data_loader.cpp`
* Live vs Backtest → Real-time stream vs historical simulation
* Can it do live trading? → Shadow only. No real orders.
* Sharpe interpretation → <0 poor, >2 excellent risk-adjusted
