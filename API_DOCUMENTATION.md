# API Documentation

---

## Strategy Development API

### Base Strategy Interface

All trading strategies must inherit from the base `Strategy` class:

```cpp
class Strategy {
public:
    virtual void on_data(const Candle& candle, const Candle* high_tf = nullptr) = 0;
    virtual bool should_buy() const = 0;
    virtual bool should_sell() const = 0;
    virtual void enable_debug(const std::string& strategy_name, const std::string& symbol) {}
    virtual void init(TradeEngine* engine) { this->engine = engine; }
    virtual ~Strategy() = default;

protected:
    TradeEngine* engine = nullptr;
};
````

#### Methods

* **`on_data(...)`**: Process new market data and update internal state.
* **`should_buy()`**: Return `true` if buy signal is active.
* **`should_sell()`**: Return `true` if sell signal is active.
* **`enable_debug(...)`**: Enable logging for debugging.
* **`init(...)`**: Initialize strategy with engine reference.

---

### Candle Structure

```cpp
struct Candle {
    std::string symbol;
    std::string timestamp_str;
    std::chrono::system_clock::time_point timestamp;
    double open, high, low, close, volume;
};
```

---

### Trade Record Structure

```cpp
struct TradeRecord {
    std::string symbol;
    std::chrono::system_clock::time_point entry_time, exit_time;
    double signal_price, entry_price, exit_price, profit;
    bool win;
};
```

---

## Strategy Examples

---

### 1. SMA Strategy

```cpp
class SMA_Strategy : public Strategy {
public:
    SMA_Strategy(int short_period = 3, int long_period = 8);
    void on_data(const Candle& low_tf, const Candle* high_tf = nullptr) override;
    bool should_buy() const override;
    bool should_sell() const override;
    void enable_debug(const std::string& strategy_name, const std::string& symbol) override;

private:
    std::deque<double> price_window;
    int short_period, long_period;
    double short_sma = 0.0, long_sma = 0.0;
    bool buy_signal = false, sell_signal = false;
    bool in_position = false;
    std::ofstream debug_log;

    double compute_sma(const std::deque<double>& prices, int period) const;
};
```

```cpp
void SMA_Strategy::on_data(const Candle& low_tf, const Candle* high_tf) {
    price_window.push_back(low_tf.close);
    if (price_window.size() > static_cast<size_t>(long_period))
        price_window.pop_front();

    buy_signal = sell_signal = false;

    if (price_window.size() >= static_cast<size_t>(short_period)) {
        short_sma = compute_sma(price_window, short_period);

        if (price_window.size() >= static_cast<size_t>(long_period)) {
            long_sma = compute_sma(price_window, long_period);
        }

        double current_price = low_tf.close;
        bool momentum_up = (short_sma > long_sma);
        bool above_sma = current_price > short_sma;

        if (!in_position && momentum_up && above_sma) {
            buy_signal = true;
            in_position = true;
        } else if (in_position && (current_price < short_sma || current_price < 0.98 * short_sma)) {
            sell_signal = true;
            in_position = false;
        }
    }
}
```

---

### 2. EMA-RSI Strategy

```cpp
class EMARSI_Strategy : public Strategy {
public:
    void on_data(const Candle& low_tf, const Candle* high_tf = nullptr) override;
    bool should_buy() const override;
    bool should_sell() const override;
    void enable_debug(const std::string& strategy_name, const std::string& symbol) override;

private:
    std::deque<Candle> candles;
    double short_ema = 0.0, long_ema = 0.0;
    double rsi = 50.0, atr = 0.0;
    double entry_price = 0.0;
    bool in_position = false;
    int cooldown = 0;
    bool buy_signal = false, sell_signal = false;

    static constexpr int short_period = 20;
    static constexpr int long_period = 100;
    static constexpr int rsi_period = 14;
    static constexpr double rsi_oversold = 30.0;
    static constexpr double rsi_overbought = 70.0;

    double ema(double prev_ema, double price, int period);
    double compute_rsi() const;
    double compute_atr() const;
};
```

---

## Trade Engine API

```cpp
class TradeEngine {
public:
    TradeEngine(double capital, Mode mode, const std::string& log_file,
                int latency_sec = 60, double slippage_bps = 5.0,
                double stop_loss_pct = 0.02, double take_profit_pct = 0.04,
                double max_drawdown_pct = 0.5);

    void update(const Candle& candle);
    void queue_buy(double price, const std::string& ts,
                   std::chrono::system_clock::time_point tp,
                   const std::string& symbol);
    void queue_sell(double price, const std::string& ts,
                    std::chrono::system_clock::time_point tp,
                    const std::string& symbol);

    const std::vector<TradeRecord>& get_trades() const;
    double get_portfolio_value() const;
    double get_realized_pnl() const;
};
```

---

## Risk Manager API

```cpp
class RiskManager {
public:
    RiskManager(const RiskLimits& limits);

    bool can_open_position(const std::string& symbol, double size, double price);
    bool can_increase_position(const std::string& symbol, double additional_size);
    void update_positions(const std::unordered_map<std::string, double>& positions);
    void update_pnl(double realized_pnl, double unrealized_pnl);

    std::vector<std::string> get_risk_alerts() const;
    bool is_risk_breach() const;

    const RiskMetrics& get_metrics() const;
    const RiskLimits& get_limits() const;
};
```

#### RiskLimits

```cpp
struct RiskLimits {
    double max_position_size = 0.1;
    double max_daily_loss = 0.02;
    double max_drawdown = 0.05;
    double var_limit = 0.03;
    double correlation_limit = 0.7;
    int max_positions = 5;
};
```

---

## Portfolio Management API

```cpp
class PortfolioManager {
public:
    PortfolioManager(double initial_capital, std::shared_ptr<RiskManager> risk_mgr);

    bool open_position(const std::string& symbol, double quantity, double price, const std::string& timestamp);
    bool close_position(const std::string& symbol, double quantity, double price, const std::string& timestamp);
    
    const Position* get_position(const std::string& symbol) const;
    std::vector<Position> get_all_positions() const;
    double get_total_portfolio_value(const std::unordered_map<std::string, double>& current_prices) const;

    PortfolioMetrics get_metrics() const;
    bool can_trade(const std::string& symbol, double quantity, double price);
    double get_cash() const;
};
```

#### Position Structure

```cpp
struct Position {
    std::string symbol;
    double quantity = 0.0;
    double avg_price = 0.0;
    double unrealized_pnl = 0.0;
    double realized_pnl = 0.0;
    std::chrono::system_clock::time_point entry_time;
    std::vector<TradeRecord> trades;
};
```

---

## Performance Analytics API

```cpp
class PerformanceAnalyzer {
public:
    static PerformanceReport analyze_strategy(const std::vector<TradeRecord>& trades,
                                              const std::vector<double>& benchmark_returns,
                                              double initial_capital,
                                              const std::string& strategy_name = "");

    static void generate_html_report(const PerformanceReport& report,
                                     const std::string& filename,
                                     const std::string& strategy_name);

    static void compare_strategies(const std::vector<std::pair<std::string, PerformanceReport>>& strategies,
                                   const std::string& output_file);
};
```

#### PerformanceReport

```cpp
struct PerformanceReport {
    double total_return = 0.0;
    double annualized_return = 0.0;
    double sharpe_ratio = 0.0;
    double sortino_ratio = 0.0;
    double calmar_ratio = 0.0;

    double max_drawdown = 0.0;
    double volatility = 0.0;
    double var_95 = 0.0;
    double var_99 = 0.0;
    double beta = 0.0;

    int total_trades = 0;
    double win_rate = 0.0;
    double avg_win = 0.0;
    double avg_loss = 0.0;
    double profit_factor = 0.0;
    double avg_trade_duration_hours = 0.0;
};
```

---

## Data Loader & Timeframe Aggregator

```cpp
std::vector<Candle> load_csv_data(const std::string& filename);
std::unordered_map<std::string, std::vector<Candle>> load_multiple_assets(const std::vector<std::pair<std::string, std::string>>& asset_files);
```

```cpp
class TimeframeAggregator {
public:
    void set_timeframes(const std::vector<std::string>& tfs);
    void add(const Candle& candle);
    const Candle* get_latest(const std::string& tf) const;

private:
    int tf_to_seconds(const std::string& tf) const;
};
```

---

## Utility

### Metrics Calculation

```cpp
class Metrics {
public:
    static MetricsResult compute(const std::vector<TradeRecord>& trades, double initial_capital, const std::string& equity_csv_file = "");
};

struct MetricsResult {
    double cumulative_return = 0;
    double max_drawdown = 0;
    double sharpe_ratio = 0;
    double sortino_ratio = 0;
    int total_trades = 0;
    double win_rate = 0;
    double avg_profit = 0;
};
```

---

### Trade Logging

```cpp
void export_trades_to_csv(const std::string& filename, const std::vector<TradeRecord>& trades);
```

---

## Configuration & Defaults

```cpp
// Engine Defaults
const double DEFAULT_SLIPPAGE_BPS = 2.0;
const int DEFAULT_LATENCY_SEC = 5;
const double DEFAULT_STOP_LOSS = 0.02;
const double DEFAULT_TAKE_PROFIT = 0.04;

// Risk Defaults
const double DEFAULT_MAX_POSITION = 0.1;
const double DEFAULT_MAX_DAILY_LOSS = 0.02;
const double DEFAULT_MAX_DRAWDOWN = 0.05;
const int DEFAULT_MAX_POSITIONS = 5;

// Strategy
const int DEFAULT_COOLDOWN_CANDLES = 10;
const double DEFAULT_VOLUME_THRESHOLD = 1.2;
```

--

Baladhithya T

baladhithyat@gmail.com
