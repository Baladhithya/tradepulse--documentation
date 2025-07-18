# TradePulse Performance Benchmarking

## Overview

This document outlines the performance benchmarking results, goals, and optimizations implemented in the TradePulse live-to-test trading platform.

---

## Benchmarking Goals

| Category              | Target Goal                               |
| --------------------- | ----------------------------------------- |
| ⚙️ Backtest Speed     | 100x real-time speed                      |
| ⏱️ Latency            | <1 ms processing per candle               |
| 📈 Throughput         | 1000+ candles/sec                         |
| 💾 Memory Usage       | Minimal RAM with sliding buffer model     |
| 🔁 Multi-threading    | Full separation of ingest vs execution    |
| 🧠 Strategy Switching | Real-time hot-swapping with zero downtime |

---

## Results Summary

### Backtesting Speed

* **BTC/USDT 1m Data** (2020–2023):

  * Processed \~1.5M candles in **8 seconds** (\~187,500 candles/sec)
  * SMA strategy: 150k/sec
  * EMA-RSI: 100k/sec

### Live Shadow Mode

* **Latency per tick (avg)**: 0.42 ms (measured using high\_resolution\_clock)
* **Max sustained rate**: 1300 ticks/sec on 4-core CPU
* **Thread split**: `Ingestion Thread`, `Strategy Thread`, `Metrics Thread`

---

## Profiling Insights

### CPU Profiling

* **Hotspots:**

  * `strategy->on_data()` (\~42%)
  * `portfolio.update_pnl()` (\~18%)
  * `trade_logger.log_trade()` (\~14%)

* **Optimizations:**

  * Inline strategy logic
  * Reuse Candle objects with object pooling
  * Reduced string allocations via `std::string_view`

### Memory Usage

* **Sliding Buffer** (Deque): Uses \~15MB per asset window for 10,000 candles
* **Dynamic Cleanup**: Automatically frees memory beyond lookback horizon
* **Symbol Map**: O(1) lookup using `std::unordered_map<std::string, AssetState>`

---

## System Design for Performance

### Data Structures

* `std::deque<Candle>` for sliding time window
* `std::vector<Order>` for fill history
* `std::unordered_map<std::string, StrategyInstance>` per asset

### Multi-Threading

* Live Mode uses:

  * **Data Feed Thread**: Handles 5s tick from Binance
  * **Strategy Thread**: Processes candle + triggers logic
  * **Metrics Thread**: Tracks CPU, RAM, order stats

* Shared state via `std::atomic`, `lock-free queues`

---

## Real-Time Monitoring

| Metric          | Method                                 |
| --------------- | -------------------------------------- |
| CPU Usage       | `getrusage` (Unix) or task manager API |
| RAM Consumption | `/proc/self/status` or WinAPI          |
| Tick Latency    | `std::chrono::high_resolution_clock`   |
| Orders/sec      | Logged in `system_monitor.cpp`         |

All logs are saved in `/logs/performance.csv` and visualized using:

```bash
python scripts/plot_performance.py  # Line + histogram plots
```

---

## Performance Tests (CI Integration)

Included in `/tests/performance_bench.cpp`:

* `TEST(BacktestSpeed)` - Verifies 100k ticks/sec minimum
* `TEST(MemoryLeakTest)` - Monitors RAM growth under load
* `TEST(LatencyBound)` - Asserts latency < 1ms per candle

---

## Future Optimizations

* GPU-accelerated metrics calculation (via CUDA/OpenCL)
* Adaptive sampling for low-volatility intervals
* Backtesting cache layer with RocksDB or Redis

---

## Authors

Performance module written by: **Baladhithya T**
