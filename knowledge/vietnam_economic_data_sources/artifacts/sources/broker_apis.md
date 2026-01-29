# Technical Guide: Vietnam Stock Broker APIs (Direct Endpoints)

This document provides technical details for high-performance direct API endpoints used by stock-tracking tools. These endpoints bypass heavy crawlers to provide low-latency market data.

## 🚀 Recommended Hierarchical Fallback
For real-time monitoring scripts (e.g., `price_alert.py`), use this prioritized hierarchy:

1.  **KBS (Primary Real-time)**: Best for active session monitoring via tick-by-tick trades.
2.  **VnDirect (Snapshot/Backup)**: Reliable price snapshots, especially on VPS.
3.  **SSI iBoard (Real-time)**: High-speed backup for live session data.
4.  **Cafef (Ajax History/Post-Session)**: Stable historical data, prone to session lag.

---

## 🏛️ 1. KBS (KB Securities Vietnam)
**Status: Primary Real-time Source**
Superior for live session monitoring where precise price matching (e.g., 23,900 vs 24,700) is required.
- **Base Referer**: `https://kbbuddywts.kbsec.com.vn/`
- **Primary Method**: `get_all_intraday_trades(symbol, max_pages=200)`
  - **Pagination Logic**: The API uses a `page` parameter (50 trades per page). Traversal continues until the `time` field reaches `09:0x` or no more data is returned.
  - **Verified Capacity**: Successfully fetched 5,000+ trades per session (ACB) for granular tape reading.
- **Key Features**: Intraday tick-by-tick (Full session), volume distribution by price, and live margin ratios.

## 🏛️ 2. VnDirect Finfo API (`finfo-api.vndirect.com.vn`)
**Best for: Reliable price snapshots and foreign flow totals.**
- **Price Endpoint**: `https://finfo-api.vndirect.com.vn/v4/stock_prices?sort=date&q=code:{symbol}&size=1`
- **Metadata Endpoint**: `https://finfo-api.vndirect.com.vn/v4/stock_prices?sort=date&q=code:{symbol}&size=2`
- **Key Fields**: `close` (Full VND), `nmVolume` (Vol), `fBuyVol` (Foreign Buy), `fSellVol` (Foreign Sell).
- **Pros**: Highly stable, 24/7 availability on VPS nodes with international routing.
- **Cons**: Rate-limited; can time out on local ISP networks.

## 🏛️ 3. SSI iBoard API (`iboard-api.ssi.com.vn`)
**Best for: Ultra-low latency and market-wide boards.**
- **Endpoint**: `https://iboard-api.ssi.com.vn/statistics/stock/{symbol}`
- **Data Shape**: Returns a JSON object with `lastPrice`, `change`, `totalVol`, `fBuyVol`, `fSellVol`.
- **Fast Board Access**: Returning the entire price board (HOSE/HNX) in a single payload is possible via the group endpoints.
- **Note**: Extremely fast but sensitive to header spoofing; requires a modern browser User-Agent.

## 🏛️ 4. Cafef Ajax Endpoint (`s.cafef.vn`)
**Best for: Session backup and post-market reconciliation.**
- **Historical Price**: `https://s.cafef.vn/Ajax/PageNew/DataHistory/PriceHistory.ashx?Symbol={symbol}&StartDate=&EndDate=&PageIndex=1&PageSize=1`
- **Units**: **Thousands of VND**. CRITICAL: Multiply by 1000 (e.g., 23.9 -> 23,900).
- **Indices API**: `https://s.cafef.vn/ajax/indexs.ashx?index=VNINDEX`
- **Lag Trap**: Prone to a 1-session delay; often returns "Yesterday's Close" during a live active session. Verify the `date` (Ngay) field in the response.

---

## 🛡️ Connectivity & Security Patterns

### 1. Headless Crawling Blockage
Broker firewalls often block the default `urllib` or `requests` User-Agent (returning 403 or empty 200).
- **Solution**: Use a desktop browser string: `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36`

### 2. IPv6 Routing Conflicts (Force IPv4)
VPS nodes (like those with FastPanel) often have misconfigured IPv6 routes causing `Address unreachable` (Exit Code 7).
- **Remedy**: Force IPv4 at the CLI (`curl -4`) or at the Python socket level to ensure sub-second resolution.

### 3. Flexible SSL Context
Internal-to-external bridging on VPS often triggers certificate validation failures.
```python
import ssl
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE  # Relaxed for broker data feeds
```

---

## 🏗️ Implementation Guidelines

### Unit Normalization Logic
Always normalize to full VND before comparison or P/L calculation.
```python
def normalize_price(price, source):
    if source == 'cafef':
        return price * 1000
    return price
```

### Blackout & Market Phase Handling
APIs may return `None` or empty lists during pre-market (8:45 AM) or ATC (2:45 PM).
- **Pattern**: Initialize with `[]`. If `len(data) == 0`, immediately poll a fallback source.
- **Stale history Check**: In monitoring scripts, always verify `if latest_date < today_date` to prevent alerting on old historical snapshots.
