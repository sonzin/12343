# Programmatic Access to Vietnam Economic Data

For building AI agents, dashboards, or analysis tools, the following APIs provide structured data.

## 🌐 World Bank Indicators API (V2)
The World Bank API is free, requires no authentication, and is highly reliable for macroeconomic indicators.

- **Base URL:** `https://api.worldbank.org/v2/`
- **Country Code for Vietnam:** `VNM` (or `VN` in some older systems, but `VNM` is standard ISO 3-letter).
- **Format:** `json` or `xml`.

### Common Indicator Codes:
- `NY.GDP.MKTP.CD`: GDP (current US$)
- `NY.GDP.MKTP.KD.ZG`: GDP growth (annual %)
- `FP.CPI.TOTL.ZG`: Inflation, consumer prices (annual %)
- `BX.KLT.DINV.CD.WD`: Foreign direct investment, net inflows (BoP, current US$)

### Example API Request:
To get Vietnam's GDP growth data in JSON format:
```
https://api.worldbank.org/v2/country/VNM/indicator/NY.GDP.MKTP.KD.ZG?format=json
```

---

## 💱 Exchange Rate & SBV Data APIs
While the State Bank of Vietnam does not provide a direct public API yet, several third-party services wrap their data.

### 1. VNAppMob
Provides an API for SBV exchange rates.
- **GET Request Example:** (Refer to VNAppMob documentation for latest endpoints)

### 2. NeoX
Another provider for VND-related FX rates.

---

## 🛠️ Python Libraries
For developers, several libraries simplify fetching this data:
- `wbdata`: A Python interface to the World Bank API.
- `yfinance`: Helpful for VN-index and publicly listed stocks on HOSE/HNX.
- `vnstock`: The most comprehensive **Python library** for Vietnam stock data. 
    - **Usage**:
      ```python
      import os
      from vnstock import Vnstock
      os.environ['VNSTOCK_HIDE_ADS'] = '1' # Suppress console ads
      # IMPORTANT: TCBS source is restricted as of Dec 15th. Use 'VCI' for public access.
      stock = Vnstock().stock(symbol='KBC', source='VCI')
      # Fetch history (most stable for 24/7 access)
      h = stock.quote.history(start='2026-01-12', end='2026-01-19', interval='1D')
      ```
    - **Note**: The `VCI` source is the current recommended public source for both historical and recently closed price data following the restriction of the TCBS public API.

---

## 📊 Data Source Selection (vnstock)

Choosing the correct source is critical for specific data requirements in the Vietnamese market:

| Feature | Source: `VCI` (Recommended) | Source: `TCBS` | Source: `SSI/VNDirect` |
|---------|-----------------------------|----------------|-----------------------|
| **Real-time Price** | ✅ Stable (Latest Close) | ❌ Restricted (12/15) | ✅ Professional (API) |
| **History (1D)** | ✅ Very Stable | ⚠️ Deprecated | ✅ Stable |
| **Foreign Flow** | ⚠️ Historical only | ❌ Unavailable | ✅ High Quality |
| **Availability** | ✅ 24/7 (Public) | ❌ Internal Only | ⚠️ Auth Required |

### 💡 Implementation Tip: The "VCI Standard" Pattern
Following the **vnstock** update on Dec 15th, the TCBS public API is no longer accessible for external scripts. The new standard for public dashboards is:
1. **Primary Source**: Use `VCI` source for `quote.history()` to get the latest daily bars. 
2. **Real-time Simulation**: Use the last bar's `close` as the current price outside of market hours or when real-time feeds are restricted.
3. **Ads Management**: Use `os.environ['VNSTOCK_HIDE_ADS'] = '1'` to clean up console output in production.
4. **Foreign Flow Disclaimer**: The `VCI` history source **does not provide real-time foreign net flow** (fBuyVol/fSellVol) for all symbols. Newly added symbols may show `0` unless the system uses a more specialized trading/metadata API (like SSI iBoard Query).

---

---

## ⚡ Fast REST Alternatives (Direct Broker APIs)

For production dashboards running on low-resource VPS or for high-frequency monitoring (like `price_alert.py`), using direct REST APIs is preferred over heavy libraries like `vnstock` to minimize latency and memory usage.

Detailed technical specs for these endpoints (KBS, SSI, VNDirect, Cafef) are consolidated in the **[Broker API Technical Guide](./sources/broker_apis.md)**.

### Key High-Frequency Patterns:
1. **Real-time Hierarchy**: KBS (Tick trades) -> VnDirect (Snapshots) -> SSI -> Cafef.
2. **Connectivity**: Use Forced IPv4 and custom User-Agents to bypass broker firewalls.
3. **Blackout Handling**: Safety wraps for 08:45 and 14:45 market transitions.

---

## 🏗️ Pattern: Local Real-time Data Proxy

For client-side web tools (dashboards) hindered by CORS, a common architectural pattern is to use a local Python script as a data bridge.

**Workflow:**
1. **Source**: Use the `UnifiedCrawler` or direct REST calls to bridge between broker data and local consumers.
2. **Proxy**: Host a Python server (e.g., using `BaseHTTPRequestHandler`) that maps the following endpoints:
    - `/api/price?symbol=KBC`: Current price (verified against hierarchy).
    - `/api/market`: Market indices and liquidity metadata.
3. **Frontend**: Use `fetch()` in JavaScript with `Access-Control-Allow-Origin: *` headers sent by the proxy.

