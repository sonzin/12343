# AI Portfolio Analysis Pattern

This document describes the implementation pattern for AI-driven portfolio analysis in the Vietnam Stock Tracker. This system combines real-time market data, technical indicators, and qualitative strategy context to provide institutional-grade advisory through LLMs (Gemini/Claude).

## 🧩 Core Components

1.  **Portfolio Manager**: Handles local storage/loading of trade positions in `data/portfolio.json` (Symbol, Entry Price, Volume, Date, Status).
2.  **Stock Data Fetcher (UnifiedCrawler)**: A sophisticated engine initialized with multiple data sources (SSI, VCI, etc.) that generates a consolidated report for each symbol including financials, price history, technical indicators, and trading flows (foreign, proprietary, insider).
3.  **Strategy Manager**: Provides macro context and specific stock ratings (e.g., "Strategy 2026") to guide the AI's "long-term" perspective.
4.  **AI Advisor**: An abstraction layer that communicates with OpenAI-compatible APIs (self-hosted or VectorEngine) with automatic fallback between models (e.g., Gemini 4.5 -> Claude 3.5).
5.  **Portfolio Analyzer**: The orchestrator (`src/ai/portfolio_analyzer.py`) that fetches all data via `UnifiedCrawler`, prepares a high-density prompt, and generates multi-channel reports (HTML + Telegram).

## 📊 Data Integration Flow

The analyzer performs an 4-step data enrichment process for each stock in the portfolio:

1.  **Technical Indicators**: RSI (14), MA20, MA50, MACD, Bollinger Bands, and ADX.
2.  **Market Sentiment**: Foreign net flow (20-day buy/sell) and Volume relative to 20-day average.
3.  **Financial Health**: P/E, P/B, and year-over-year income/balance sheet trends.
4.  **Macro Alignment**: Labels the stock based on its inclusion in sector themes (e.g., "Industrial Real Estate", "Banking Revolution").

## 🤖 Prompt Engineering Strategy

The system uses a "Role-Conditioned" prompt:
- **System Prompt**: Defines the persona (Vnstock Expert with 20 years experience) and specific decision rules (e.g., "MUA GIA TĂNG if Price > MA20 & RSI < 70").
- **User Prompt**: Supplies a raw JSON dump of the enriched data, market context (VN-Index), and current P/L status.
- **Goal**: Transmit maximum data density in a single context window to minimize halluncinations about price levels.

## 🛠️ Implementation Schema (Python)

```python
class PortfolioAnalyzer:
    def analyze_portfolio(self):
        # 1. Load symbols
        positions = self.manager.load_portfolio()
        
        # 2. Enrich with real-time data
        enriched_data = []
        for pos in positions:
            data = self.fetcher.fetch_all(pos['symbol'])
            enriched_data.append(self._calculate_metrics(pos, data))
            
        # 3. Request AI Advisory
        report = self._get_ai_analysis(enriched_data, market_data)
        
        # 4. Dispatch Reports
        self.reporter.generate_html(enriched_data, report)
        self.telegram.send(report)
```

## ⚠️ Robustness & Troubleshooting

### 1. Guarding against `NoneType` errors
When fetching data from public APIs (SSI/VCI), fields may be missing during "Blackout" periods (market open/close transitions). 
- **Pattern**: Use `.get(key, 0)` or explicit `if val is not None` checks before performing arithmetic or formatting. 
- **F-string trap**: `f"{val:.2f}"` will crash if `val` is `None`. Use `f"{val or 0:.2f}"` or a helper formatter.

### 2. Lazy-Loading Initialization (The StrategyManager Case)
A common architectural trap in Python is the "Early Return Failure" in lazy-loading methods.

- **The Bug**:
  ```python
  def load_context(self):
      if not self.file.exists():
          return "File missing" # Returns string, but self.cache remains None
      self.cache = self.file.read()
      return self.cache

  def check_symbol(self, symbol):
      if not self.cache: 
          self.load_context()
      # CRASHES here with 'NoneType is not iterable' if file was missing
      if symbol in self.cache: 
          ...
  ```
- **The Fix (Stateful Initialization)**: Ensure the cache variable is assigned a default value (e.g., an empty string or the error message) in *all* logical paths, including early returns and exception blocks.
  ```python
  def load_context(self):
      if not self.file.exists():
          self.cache = "File missing" # Initialize the state variable!
          return self.cache
      ...
  ```

### 3. Multi-Model Fallback
To ensure 24/7 availability even during API outages:
```python
# Fallback Chain Pattern
FALLBACK_MODELS = [
    "gemini-claude-sonnet-4-5-thinking",
    "gpt-4o",
    "gemini-3-pro-preview"
]
```

### 3. Special Character Encoding & Rendering
When constructing HTML or Telegram messages, ensure all Vietnamese characters and financial symbols (📈/📉) are UTF-8 encoded. When using `requests.post` to Telegram, use `json={'text': msg, 'parse_mode': 'HTML'}` to handle HTML tags correctly.

- **Markdown Rendering Bug**: The custom `_markdown_to_html` parser may fail on nested blockquotes (e.g., `> > Content`). 
  - *Symptom*: HTML shows raw `> >` symbols or breaks the layout.
  - *Fix*: Ensure the regex or split logic recursively strips blockquote markers or use a standardized markdown-it/markdown library if environment allows.
  - **Table Layout**: In the web report, if markdown tables are rendered as text, verify that the `table-container` CSS and the parser's logic for detecting `|` delimiters are correctly handling whitespace.

### 4. Direct Environment Execution
In modern setup iterations, the project favors direct execution over isolated `venv` environments for scripts within the `src/` directory to simplify cross-platform synchronization (Mac vs. VPS).
- **Execution Command**: `python3 src/ai/portfolio_analyzer.py`
- **Dependency Note**: Legacy scripts in `scripts/` (e.g., `portfolio_monitor.py`) may still expect `venv` or deprecated packages like `vnstock_data`. The `src/` based tools use the robust `UnifiedCrawler` which centralizes dependency management through its own unified interface.
- **Data Persistence**: Portfolio updates are made directly to `data/portfolio.json` with fields: `symbol`, `entry_price`, `volume`, `entry_date`, `note`, `status` (active/pending).

### 5. Interactive CLI & Alerting Tools
For manual interaction and automated monitoring:
- **Interactive CLI**: `python3 cli_interactive.py` allows for manual portfolio updates and scanner triggers.
- **Price Alert Monitor**: `python3 scripts/price_alert.py` is a VPS-optimized tool that scans the portfolio during trading hours.
  - **Evolution**: Originally based on `UnifiedCrawler`, it was refactored for speed. It uses a **multi-source fallback hierarchy** (KBS -> VnDirect -> SSI -> Cafef) to fetch raw price data without the overhead of full data reports. This shift was necessary because high-frequency 1-minute scans on a VPS require minimal latency and resilient failovers.
  - **Real-Time Accuracy**: By prioritizing **KBS intraday trades**, the monitor captures tick-by-tick prices (e.g., live 23,900) instead of session-lagging summaries (e.g., 24,700).
  - **Capabilities**: Sends Telegram alerts for Stop-Loss (-5%), Take-Profit (+10%), Warning (-3%), and \"Pending Match\".
  - **Urgent vs. Periodic Alerting**: 
    - **Stop-Loss**: Configured for **continuous alerting** (every scan) when triggered. In the code, this is achieved by removing the `should_alert` conditional check for the `STOP_LOSS` alert type and omitting `mark_alerted`.
    - **Others (Warning/TP)**: Follow the **Once-per-Day** rule using `if should_alert(...)` to prevent notification fatigue while still providing daily visibility.
  - **Operation Flags**:
    - `-v, --verbose`: Enables debug mode, showing all ticker checks and price comparisons even if no alert is triggered.
    - `--force`: Bypasses the "is_trading_time" check for testing or weekend analysis.
  - **Efficiency**: Remains silent unless a price threshold is reached and uses `data/alert_state.json` to manage alert history. While most alerts are suppressed after the first daily send, critical Stop-Loss triggers can be exempted from this state-check to maintain urgency.

### 6. Report Deployment & Viewing
- **Local Viewing (Mac)**: If auto-upload fails, use the `open` command to view HTML reports locally:
  ```bash
  open data/reports/portfolio_YYYYMMDD_HHMMSS.html
  ```
- **Vercel Integration**: To enable cloud reports (`REPORT_BASE_URL`), the environment requires Vercel CLI:
  ```bash
  npm install -g vercel
  vercel login # Account: quangsoncenstaf-2994
  ```
- **Automated Deployment**: The `deploy_reports.py` script automates the process by running `vercel --yes --prod` from within the reports directory. 
- **Production URL**: [https://reports-sandy-six.vercel.app](https://reports-sandy-six.vercel.app)
- **Local Cleanup Pattern**: After a successful cloud push, it is recommended to delete local `.html` reports in `data/reports/` to keep the local workspace/Drive storage lean, as the history is preserved on the Vercel deployment.

### 7. Data Robustness & Crawler Issues

- **Zero Liquidity Failure & Unit Trap**: Occurs during "Blackout" periods (11:30-13:00) or when primary data sources fail.
  - **The "Billions" Trap**: APIs like SSI often return `total_value` already in **Billions VND** (tỷ VND). A common developer error is dividing by `1e9` again, resulting in a near-zero or zero value in reports. 
  - **Fixed Pattern**: Keep `liquidity` as return value directly if the source is already scaled.
- **Foreign Trading Flow (Market-wide)**: Real-time industry aggregates often omit market-wide net flow for the Index. 
  - **Pattern**: Manually calculate `foreign_net` as `foreign_buy_value - foreign_sell_value` from the index-level data provided by the crawler. Double check units (usually Millions/Billions VND).
  - **AI Mitigation**: If global metrics remain unreliable, the system shifts focus to stock-specific indicators (RSI, Vol/MA20) and individual ticker net flows.
- **The "Real-Time Price" Trap**: When using `UnifiedCrawler.get_full_report(symbol)`, the `quote` dictionary might return empty for certain tickers or during specific market phases.
  - **Symptoms**: Script reports "Price: 0" or incorrect P/L of -100%.
  - **Solution (Multi-Source Fallback)**: For high-frequency monitoring, use a prioritized hierarchy of direct public endpoints. Note that VPS environments often have better routing/access to these endpoints than local ISP networks:
    1.  **KBS (Intraday)**: `KBSCrawler.get_intraday_trades(symbol)`. (Most accurate real-time price from tick data).
    2.  **VnDirect (Snapshots)**: `finfo-api.vndirect.com.vn`. (Reliable fallback on VPS; returns full VND units).
    3.  **SSI iBoard**: `iboard-api.ssi.com.vn`. (Alternative real-time stats).
    4.  **Cafef (Ajax)**: `s.cafef.vn`. (Session backup; prone to 1-day lag and returns thousands of VND).
  - **Unit Consistency & Multipliers**: 
    - **Cafef**: Returns prices in **thousands of VND** (e.g., 24.7). Must multiply by **1000** (e.g., `24.7 * 1000 = 24,700`).
    - **VnDirect/SSI**: Usually return **full VND units**. Always verify the first response manually.
- **The "Stale History" Trap**: Certain methods like `get_price_history()` (without specific date parameters) may return very old historical data (e.g., from **2020**) instead of the current day's price.
  - **Symptoms**: P/L calculations are wildly inaccurate but the price value looks "valid" at first glance.
  - **Prevention**: Always inspect the `date` field in the returned price object. In `price_alert.py`, prefer methods that aggregate the latest active market data or explicitly check `if latest_date < today`.
- **The "Historical Endpoint Lag" Trap**: Historical data endpoints (like Cafef's `PriceHistory.ashx`) are often committed *after* the daily trading session completely settles and may lag behind the actual live tape.
  - **Symptoms**: The monitor reports a price that matches the previous day's close (e.g., 24,700), resulting in an inaccurate P/L (e.g., -1.2%) while the real-time price is significantly different (e.g., 23,950 / -4.2%).
  - **Solution**: For high-stakes alerting, always verify the `date` field (e.g., `Ngay` in Cafef results) against the current calendar date. If the date is not today, the data is "Stale Snapshot" and should be ignored or flagged as unreliable in logs.
- **Stale Report Deployment**: If the user updates `portfolio.json` but the web report still shows old symbols (e.g., deleted ticker PTB still appears), check:
  1. **Crawler Loop**: Ensure the symbol list is refreshed from disk *inside* the execution loop, not just at startup if running in a long-lived process.
  2. **Vercel Cache**: Deployments are immutable, but ensuring the `deploy_reports.py` script targets the correct `data/reports` directory is critical.
  3. **Symbol Uniqueness**: If multiple files are deployed simultaneously, verify the URL timestamp matches the latest execution.
- **VPS Connection Block (SSH/SFTP)**: Using `rsync` or SFTP to deploy scripts to the VPS may return `Connection refused` on Port 22.
  - **Diagnostic**: This typically indicates that Port 22 is either disabled in `sshd_config`, blocked by a firewall (UFW/Iptables), or requires an alternative port (e.g., CloudFlare Proxy or custom ISP port).
  - **Solution**: Use `-p <port>` in SSH/rsync commands or check VPS provider firewall settings (Security Groups).
