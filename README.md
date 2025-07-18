# TradePulse - Live Strategy Backtester

## Overview

TradePulse is a high-performance Live-to-Test Strategy Pipeline that enables seamless development, backtesting, and deployment of trading strategies. The system provides a comprehensive framework for strategy development, rigorous backtesting with realistic market conditions, and real-time performance tracking in a live shadow environment.

## Key Features

- **High-Performance Backtesting Engine**: Event-driven simulation with realistic market conditions
- **Live Shadow Environment**: Real-time strategy execution without capital risk
- **Multi-Asset Support**: Trade across multiple cryptocurrency pairs simultaneously
- **Advanced Risk Management**: Real-time position monitoring and risk controls
- **Strategy Optimization**: Parameter optimization with walk-forward analysis
- **Market Regime Detection**: Adaptive strategies based on market conditions
- **Comprehensive Analytics**: Performance attribution and risk metrics

## System Architecture

\`\`\`
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Data Layer    │    │  Strategy Layer │    │ Execution Layer │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • Data Loader   │    │ • SMA Strategy  │    │ • Trade Engine  │
│ • CSV Parser    │    │ • EMA-RSI       │    │ • Order Queue   │
│ • Live Feeds    │    │ • Momentum      │    │ • Slippage      │
│ • Timeframe     │    │ • Breakout      │    │ • Latency       │
│   Aggregator    │    │ • Volatility    │    │ • Portfolio Mgr │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
┌─────────────────────────────────┼─────────────────────────────────┐
│                    Analytics & Risk Layer                        │
├─────────────────────────────────┼─────────────────────────────────┤
│ • Performance Analyzer  • Risk Manager  • System Monitor        │
│ • Market Regime Detector • Strategy Optimizer                   │
└─────────────────────────────────────────────────────────────────┘
\`\`\`

## Quick Start

### Prerequisites
- C++17 or higher
- CMake 3.10+
- Python 3.7+ (for analysis scripts)

### Build Instructions

\`\`\`bash
# Clone and build
mkdir build && cd build
cmake ..
make -j4

# Run backtesting
./tradepulse

# For live shadow trading
python scripts/binance_stream.py 1  # Start BTC data stream
./tradepulse  # Select live shadow mode
\`\`\`

## Supported Strategies

1. **SMA Strategy**: Simple Moving Average crossover with aggressive parameters
2. **EMA-RSI Strategy**: Exponential Moving Average with RSI confirmation
3. **Momentum Strategy**: Price momentum with volume confirmation
4. **Breakout Strategy**: High/low breakout with volume and ATR filters
5. **Volatility Expansion**: ATR-based volatility breakout strategy

## Performance Metrics

The system tracks comprehensive performance metrics:
- **Return Metrics**: Total return, Sharpe ratio, Sortino ratio
- **Risk Metrics**: Maximum drawdown, VaR, volatility
- **Trade Metrics**: Win rate, profit factor, average trade duration
- **System Metrics**: Processing speed, latency, resource usage

## Directory Structure

\`\`\`
tradepulse/
├── src/                    # Core C++ source files
├── include/                # Header files
├── scripts/                # Python analysis scripts
├── data/                   # Historical market data
├── build/                  # build file
    ├── logs/                   # Trade logs and debug files
    ├── reports/                # Generated HTML reports
    ├── plots/                  # Performance charts
    └── docs/                   # Documentation
└── CMakeLists.txt          # CMake build configuration
\`\`\`

## License
Done by : 
Baladhithya T

baladhithyat@gmail.com

+91 9751418918
This project is developed for the GoQuant recruitment process.
\`\`\`
