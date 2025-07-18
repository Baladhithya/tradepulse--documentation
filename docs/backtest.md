# Backtesting Methodology - TradePulse

## Overview

The backtesting system in TradePulse simulates historical trading conditions to evaluate strategy robustness. It is engineered to closely replicate real-world market dynamics, including latency, slippage, transaction costs, and market microstructure.

---

## Event-Driven Architecture

TradePulse uses an event-driven simulation model:

1. **Data Feed Ingestion**

   * Reads candle data from CSV files or memory streams
   * Supports multiple assets and timeframes (e.g., 5s, 1m, 5m, 1h)

2. **Strategy Trigger**

   * On each candle event, `Strategy::on_data()` is called
   * Signals (`should_buy()`, `should_sell()`) are evaluated

3. **Order Queue & Execution**

   * Orders are queued with optional latency delay
   * Simulated slippage and execution mechanics

4. **Portfolio Update**

   * P\&L, position sizing, and logs are updated in real time

5. **Performance Metrics Recording**

   * Trades, signals, and results are written to logs and CSVs

---

## Assumptions & Constraints

### Market Conditions

* **No perfect fill**: Slippage and bid/ask spread modeled
* **No lookahead bias**: Strategies only use past or current candles
* **Execution delay**: Configurable latency added before trade fills

### Time Granularity

* Engine supports fine-grained intervals (e.g., 5s candles)
* Aggregator rebuilds higher timeframes from base resolution

### Capital Assumptions

* Default starting capital: \$100,000
* Position sizing defined in strategy or risk module

---

## Realism Enhancements

### Slippage Modeling

```cpp
price *= (is_buy ? (1.0 + slippage_rate) : (1.0 - slippage_rate));
```

* Default: 0.02% per trade (can be strategy-specific)

### Latency Injection

```cpp
std::chrono::seconds latency = std::chrono::seconds(5);
order.execution_time = now + latency;
```

* Simulates network and exchange delay

### Transaction Cost Handling

* Fixed spread and % slippage simulated
* Optional: integrate commission model in future

---

## Multi-Asset Support

Backtester can run multiple assets in parallel:

```cpp
for (auto& [symbol, candle_stream] : asset_candles) {
    auto strategy = factory();
    strategy->init(engine);
    for (const auto& candle : candle_stream) {
        strategy->on_data(candle);
    }
}
```

* Isolated positions, logs, and metrics per asset
* Cross-asset strategy logic also supported

---

## Validation Techniques

### Walk-Forward Testing

* Splits data into rolling train/test windows
* Validates generalization capability

### Monte Carlo Simulation

* Randomizes order execution sequences
* Measures performance distribution under uncertainty

### Out-of-Sample Testing

* Uses held-out periods for final validation

---

## Output Artifacts

* `/logs/trades.csv`: All trade records
* `/logs/signals.csv`: Buy/sell signal timestamps
* `/logs/performance.csv`: Metrics over time
* `/plots/`: Visualized P\&L, drawdown, exposure

---

## Future Enhancements

* Commission models per exchange
* Partial fills and slippage curve modeling
* Order book simulation with L2 data
* Multi-threaded batch backtesting
* GUI to visualize live backtest progress
