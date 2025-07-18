

# **TradePulse - Live Strategy Backtester**

## **Overview**

**TradePulse** is a high-performance Live-to-Test Strategy Pipeline that enables seamless development, backtesting, and deployment of trading strategies. It offers a comprehensive framework for:

* Strategy development
* Realistic backtesting
* Live shadow trading with no capital risk
* Performance monitoring

---

## **Key Features**

* **⚡ High-Performance Backtesting Engine**
  Event-driven simulation with realistic market microstructure.

* **🟢 Live Shadow Environment**
  Real-time strategy execution without using real capital.

* **📊 Multi-Asset Support**
  Trade across multiple crypto pairs (e.g., BTC/USDT, ETH/USDT, etc.).

* **🛡️ Advanced Risk Management**
  Dynamic position sizing and portfolio-level risk limits.

* **🧠 Strategy Optimization**
  Walk-forward analysis and parameter sweeps.

* **📈 Market Regime Detection**
  Adapt strategy logic based on volatility, momentum, or trend regimes.

* **📉 Performance Analytics**
  Comprehensive metrics: returns, drawdown, Sharpe, Sortino, VaR, alpha/beta attribution.

---

## **System Architecture**

![TradePulse Architecture](https://github.com/user-attachments/assets/3033de4d-8e40-4563-aba2-ed08f12f588a)

### **Modules**

* **Data Ingestion:** Historical CSV or live feed (Binance WebSocket)
* **Timeframe Aggregator:** Resamples data from 5s to 1m, 5m, etc.
* **Strategy Engine:** Plug-and-play strategy execution
* **Risk Manager:** Enforces max drawdown, position limits, and scaling
* **Trade Logger:** Persists every trade, signal, and fill
* **Performance Analyzer:** Evaluates metrics in real-time
* **Market Regime Detector:** Adjusts logic based on volatility & trend

---

## **Quick Start**

### **Prerequisites**

* C++17 or later
* CMake ≥ 3.10
* Python 3.7+ (for real-time streaming)

---

### **Build Instructions**

```bash
# Clone repository
git clone https://github.com/your-org/tradepulse.git
cd tradepulse

# Build the C++ project
mkdir build && cd build
cmake ..
make -j4

# Run backtest mode
./tradepulse

# Start live shadow trading
python scripts/binance_stream.py 1  # Start 5s Binance BTC feed
./tradepulse  # Choose 'live' mode in CLI
```

---

## **Supported Strategies**

| Strategy             | Description                                       |
| -------------------- | ------------------------------------------------- |
| SMA Crossover        | Simple moving average cross with fast/slow params |
| EMA-RSI              | EMA crossover confirmed by RSI filters            |
| Momentum Strategy    | Momentum rank with volume spike confirmation      |
| Breakout Strategy    | High/low breakout with ATR and volume filters     |
| Volatility Expansion | Breakout strategy based on volatility bursts      |

Strategies are modular and can be registered via `strategy_registry.hpp`.

---

## **Performance Metrics**

| Category       | Metrics Tracked                                 |
| -------------- | ----------------------------------------------- |
| Return Metrics | Total Return, Annualized Return, CAGR           |
| Risk Metrics   | Max Drawdown, Sharpe, Sortino, Value-at-Risk    |
| Trade Metrics  | Win Rate, Profit Factor, Avg Duration, Avg R\:R |
| System Metrics | Orders/sec, Latency, CPU/RAM Usage              |

All results are saved in `/logs/performance.csv` and can be visualized via Python.

---

## **Directory Structure**

Architecture visual aid:

![Directory Structure](https://github.com/user-attachments/assets/44e21f74-53e6-4e8d-b4ae-5645f6bc2a06)

---

## **License & Credits**

Developed by:

**Baladhithya T**
📧 [baladhithyat@gmail.com](mailto:baladhithyat@gmail.com)
📱 +91 9751418918

This project was developed as part of the **GoQuant Recruitment Process**.

---

