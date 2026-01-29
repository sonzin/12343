# Tool Pattern: Vietnam Stock Tracker Dashboard

This document describes the architectural pattern and implementation of a client-side Vietnam Stock Tracker Dashboard. This tool is designed for retail investors to monitor their portfolio, track market sentiment (foreign flows), and manage trading plans with automated risk alerts.

## 🏗️ Architecture

- **Frontend**: Vanilla HTML5, CSS3 (Modern Dark Theme), and JavaScript (ES6+).
- **Backend (API Server)**: Python-based HTTP server using `vnstock` (VCI source) or direct REST clients to serve market data as JSON.
- **Data Persistence**: `localStorage` (Client-side browser storage) for portfolio data, with the backend serving as a transient data proxy to bypass CORS.
- **Hygiene**: Console ad suppression via `os.environ['VNSTOCK_HIDE_ADS'] = '1'`.
- **State Management**: Simple JavaScript object-based state synced with local storage.
- **UI Paradigm**: Glassmorphism with responsive grid layouts.

## 🚀 Key Features

### 1. Portfolio Tracking
- **P&L Calculations**: Real-time calculation of Profit and Loss in both percentage and absolute value based on entry price and current price.
- **Automatic Multipliers**: Handles quantity and price to provide total valuation of holdings.

### 2. Risk Management
- **Dynamic Alerts**: Visual and auditory alerts when stock prices hit user-defined **Stop-Loss** or **Take-Profit** thresholds.
- **Visual Levels**: Color-coded badges for SL and TP levels on each stock card.

### 3. Market Sentiment Integration
- **Foreign Flow Bar**: Visual representation of foreign buy vs. sell volume.
- **Momentum Indicators**: Tracking volume spikes and price action relative to daily highs/lows.

### 4. Qualitative Analysis
- **Note-taking System**: Integrated markdown-style notes for each stock to record catalysts, research findings, and rating (Bullish/Neutral/Bearish).

### 5. Resilient Real-time Connectivity
- **After-hours Logic**: Automatically falls back to fetching the previous day's close (Historical Data) if the real-time Price Board is unavailable (e.g., market is closed).
- **Auto-Refresh**: Configurable polling interval (e.g., 30s) that automatically halts outside of HOSE/HNX trading hours to conserve resources.
- **Connection Status**: Live indicators for "API Online/Offline" and "Market Open/Closed".

## 🛠️ Implementation Schema

### Data Model (Stock Object)
```javascript
{
    id: "uuid",
    symbol: "KBC",
    entryPrice: 38000,
    currentPrice: 38000,
    quantity: 1000,
    stopLossPercent: 8,
    takeProfitPercent: 12,
    foreignBuy: 2924100,
    foreignSell: 787000,
    volume: 20477000,
    note: "Description here",
    updatedAt: "ISO-DATE"
}
```

## 💡 Best Practices for Usage
- **Real-time Mode**: Run the `stock_api.py` server locally to enable automatic price updates every 30-60 seconds during trading hours.
- **Data Batching**: Use a single `/api/prices?symbols=A,B,C` request instead of individual calls to optimize performance and reduce broker rate-limiting.
- **CORS Handling**: The Python server includes headers to allow cross-origin requests from the local file system (where the HTML is opened).
- **Risk Ratios**: The dashboard encourages a 1:2 or 1:1.5 Risk/Reward ratio by pre-calculating targets based on entry.
- **"Shark" Detection**: Use the integrated foreign flow bars and volume statistics to verify breakouts against the manipulation patterns discussed in economic analysis artifacts.
- **V3.1 Implementation**: Reverted to `vnstock` with **VCI source** for maximum stability on local machines. Note that VCI returns `0` for foreign buy/sell volume when using simple history fetches; this requires specialized metadata APIs for full functionality.
- **Console Hygiene**: Suppress vnstock startup messages in the API server logs using `os.environ['VNSTOCK_HIDE_ADS'] = '1'`.
- **Production Port Mapping**: For robust isolation on shared servers (e.g., with FastPanel), use port `9696` for the public Nginx entry point and `9797` for the internal Python backend.
- **IP & Timeout Management**: VPS environments with shared IPs are prone to being blocked by broker firewalls (causing timeouts). V3.0 handles this by using lightweight `urllib` requests with 10s timeouts to prevent the dashboard from hanging.

## 📊 Data Considerations & Field Nuances

When integrating real-time data for the Vietnamese market, developers must account for common user confusion regarding data "staleness":

### 1. Market Hours Awareness
- **Trading Window**: HOSE/HNX operates 9:00 AM - 11:30 AM and 1:00 PM - 3:00 PM (GMT+7).
- **Staleness**: Outside these hours, current price APIs often return the final close price. The dashboard should clearly indicate "Market Closed" to prevent users from thinking the data is "broken" because it isn't moving.

### 2. Volume & Flow Tracking
- **Total Volume**: Includes all matched orders.
- **Foreign Flow (khối ngoại)**: Tracking buy/sell volume of foreign investors is a key sentiment indicator in Vietnam. 
- **Limitation**: While real-time sources (TCBS) provide "Net Buy/Sell" during the session, historical fallbacks often only provide "Total Volume." To track buy/sell volume accurately 24/7, the system must cache and store the session's final foreign flow data before the real-time API goes offline at market close.

### 3. Detailed Volume & Flow Tracking
For professional analysis, users require tracking of:
- **Foreign Net Flow**: Helps detect if prices are being driven by institutions or retail speculation.
- **Volume Momentum**: Comparing current volume to the 20-day average. In many automated tools, this is displayed as a percentage (e.g., "Volume is at 80% of daily Avg").
- **Real-time Availability (The "Blackout")**: If a user observes "0" for volume or flow, it usually indicates the backend has transitioned to an "After-hours" source or the public API (SSI/Cafef) is currently returning empty arrays due to the market being closed. To maintain data continuity, systems should implement a persistent cache that "freezes" the last known session values at 2:45 PM.

### 3. Price Precision
- Stock prices in Vietnam are often quoted in thousands (e.g., `38.0` for 38,000 VND). The backend proxy should normalize these values to full units to simplify frontend calculations.

### 4. Fast API Resilience
For production success:
- **Direct REST over Libraries**: Use `urllib` to hit `https://iboard.ssi.com.vn/dchart/api/1.1/defaultAllStocks` to avoid library overhead.
- **Multi-Source Fallback**: Configure the API to try the SSI iBoard API first, then fallback to Cafef's `quote.ashx` for missing metadata.

## 🌐 Production Deployment

For deploying the tracker to a remote server (e.g., Ubuntu VPS), the following production architecture is recommended:

### 1. Reverse Proxy (Nginx)
- **CORS Handling**: Move CORS logic from the application to the Nginx layer or use Nginx to serve both the frontend and API from the same origin.
- **SSL/TLS**: Terminate SSL at Nginx using Let's Encrypt for secure data transmission.
- **Compression**: Enable Gzip for faster loading of static assets (JS/CSS).

### 2. Process Management (Systemd)
- **Persistence**: Use `systemd` to manage the Python API server. This ensures the server restarts automatically on failure or system reboot.
- **Security**: Run the service under a dedicated, non-root system user (e.g., `stocktracker`) with restricted permissions to the application directory.

### 3. CI/CD Patterns
- **Automated Provisioning**: Use shell scripts to automate the installation of dependencies (`python3-venv`, `pip`), user creation, and service configuration.
- **Deployment Automation**: Tools like `sshpass` can be used in simple deployment scripts to automate the `scp`/`ssh` workflow while maintaining a separation between local development and production environments.

## 🛠️ Troubleshooting & Developer Hygiene

When running custom API servers locally for extended periods, developers often encounter specific environmental friction:

### 1. Port Conflicts (Address Already in Use)
The Python server binds to a specific port (e.g., `8888`). If the script is restarted without properly killing the previous instance, you will receive `OSError: [Errno 48]`. 
- **Identify Blocking Process**: `lsof -ti:8888`
- **Kill Conflict**: `lsof -ti:8888 | xargs kill -9`
- **Automated Hygiene**: Incorporate a cleanup command into your launch script: `pkill -f "stock_api.py" 2>/dev/null || true`.

### 2. Dependency Evolution
`vnstock` is a fast-evolving library. Warnings like `Could not patch KBS Finance` or metadata mismatches often occur after a broker updates their internal API. 
- **Remedy**: Regularly run `pip install --upgrade vnstock` to synchronize with the latest scraper patches.
- **Environment Isolation**: Always use a `venv` (Virtual Environment) to prevent conflicts between the stock tracker's specific data source needs and other Python projects.

