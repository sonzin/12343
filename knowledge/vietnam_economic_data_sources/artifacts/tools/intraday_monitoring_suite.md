# Intraday Monitoring and Risk Management Suite

This suite of tools provides multiple layers of intraday stock market analysis, ranging from raw data extraction to strategic behavioral intent reading. The goal is to act as a **"Risk Manager"** rather than a high-frequency trader.

## 🛠️ Overview of Tools

| **Raw (Legacy)** | `scripts/export_intraday.py` | Full data extraction | Data archiving, spreadsheet analysis. |
| **Smart (v1.1)**| `scripts/smart_intraday.py` | Behavioral intent | Reading market rhythm via session-blocks, VNINDEX, and institutional context. |

---

## 1. Smart Intraday Analyzer (`smart_intraday.py`)

The "Pro" level tool that implements the **Optimal Monitor Combo**. It aggregates data into psychological session blocks.

### 📋 Key Features
- **Session-Block Aggregation**: Breaks the day into psychological segments: ATO (Opening intent), Morning Sprints (1-3), Afternoon Sprints (1-2), and ATC (Closing intent).
- **Automated Behavioral Observations**: The tool automatically detects:
    - **Opening Pressure**: Identifying active dumping vs. positive absorption in the first 15 mins.
    - **Closing Intent**: Flagging institutional defense or exit in the final auction (ATC).
    - **Big Order Tracking**: Distinguishing institutional "footprints" (>30k shares) from retail noise.
- **Optimal Monitor Combo (The "Standard Feed" Logic)**: Aggregates a 3-layered data hierarchy designed for AI analysis:
    - **Layer 1 (Core)**: Intraday summary (Open/Close/High/Low) and Session Flows.
    - **Layer 2 (Market Context)**: **VNINDEX** performance (Price, Vol, Width) and stock **Vol vs. 20D Average** ratios.
    - **Layer 3 (Institutional)**: Net Foreign and Net Proprietary trading flows + Personal Position Context (Entry price, status, notes).
- **Professional Feed (JSON v1.1)**: Generates a high-resolution structured JSON featuring:
    - **`day_summary`**: Automated classification (e.g., `DISTRIBUTION_TO_ACCUMULATION`) by comparing Morning/Afternoon flows.
    - **`key_levels`**: Dynamic Support and Resistance identified through volume-weighted price distribution (Liquidity Zones).
    - **`trader_context`**: Automated decision-bias (`HOLD`, `REVIEW_STOPLOSS`, `WAIT_CONFIRM`) calculated by merging P/L and behavioral intent.

### 🚀 Usage
```bash
# Analyze ACB (full session by default)
python3 scripts/smart_intraday.py ACB

# Analyze with console print
python3 scripts/smart_intraday.py ACB -p

# Fast analysis (latest 50 trades only)
python3 scripts/smart_intraday.py ACB --latest
```

---

## 2. Intraday Data Exporter (`export_intraday.py`)

The legacy engine for harvesting high-resolution tick data. Note: Most of this functionality is now superseded by `smart_intraday.py` for day-to-day decision making.

The engine for harvesting high-resolution tick data.

### 📋 Key Features
- **Full-Session Pagination**: Traverses the KBS API (50 trades/page) to reconstruct the entire session history (6,000+ trades for active stocks).
- **Multi-Format Output**: JSON (programmatic) and CSV (manual) exports saved to `data/intraday/`.
- **Chronological Normalization**: Reverses latest-first API results into a 09:00 → 15:00 timeline.

### 🚀 Usage
```bash
# Full export to data/intraday/
python3 scripts/export_intraday.py ACB

# Quick export (latest page only)
python3 scripts/export_intraday.py ACB --latest
```

---

## 📉 Strategic Application: The "Optimal Monitor Combo"

To avoid emotional bias, use the outputs of these tools to identify three specific market states:

1. **Shakeout (Rũ hàng)**: Price Down + VNINDEX Stable + Vol < 70% Avg.
   - *Reaction*: Hold. This is noise or a bear trap.
2. **Controlled Distribution**: Price Stable + Vol > 120% Avg + Institutional Selling.
   - *Reaction*: Tighten Stop-loss. Professionals are exiting without panicking the crowd.
3. **Absorption Defense**: Price at Support + High BUY % in Session Blocks + Big Order support.
   - *Reaction*: Positive sign. Setup is being defended by "Smart Money."

---

## 3. Standardized Output Architecture

The suite uses a unified output structure to minimize clutter and ensure data consistency:

- **Primary Destination**: `data/intraday/`
- **Standardized Files**:
    - `[SYMBOL]_smart_[YYYYMMDD].md`: The Human/AI-ready report (Standard Feed).
    - `[SYMBOL]_smart_[YYYYMMDD].json`: High-resolution data feed for programmatic analysis and ChatGPT optimization (v1.1).
    - `[SYMBOL]_intraday_[YYYYMMDD]_full.json`: Raw tick data preserved for debugging or deep auditing.

### 🧹 Directory Management
Legacy folders like `data/intraday_monitor/` and `data/intraday_smart/` have been phased out. All active trading intelligence is consolidated into the `data/intraday/` directory.

---

## 🧠 Strategic Heuristics (Pro Logic v1.1)

The Smart Analyzer uses algorithmic logic to quantify market psychology:

### 1. Day Type Classification (`day_type`)
Determines the intraday "shape" of volume and price:
- `DISTRIBUTION_TO_ACCUMULATION`: Morning selling pressure followed by successful afternoon absorption (The "U-shape" or "V-shape").
- `ACCUMULATION_TO_DISTRIBUTION`: Morning strength that fails into afternoon liquidations.
- `ACCUMULATION`: Consistent buying pressure.
- `DISTRIBUTION`: Consistent selling pressure.

### 2. Automated Action Biases (`action_bias`)
Calculated by merging position state (Active/Pending) with intraday flow:
- **WAIT_CONFIRM**: Recommended during "Distribution to Accumulation" phases or as a safeguard during **Distribution** days for pending orders.
- **REVIEW_STOPLOSS**: Triggered when a stock is in a "Distribution" state and P/L is below -3%.
- **CONSIDER_TAKE_PROFIT**: Triggered on "Distribution" states when P/L is positive (>3%).
- **READY_TO_BUY / BUY_SIGNAL**: Triggered for pending orders when price targets are hit and the flow is supportive (Accumulation).

### 3. Finalized Risk Modes (`risk_mode`)
- **SAFE**: Strong trend with positive P/L.
- **WARNING / DANGER**: Active positions hitting risk thresholds.
- **OPPORTUNITY**: Pending positions at or below target price with valid setups.
- **WATCH**: Pending positions approaching targets (within 2% buffer).
- **NEUTRAL**: Stable positions or pending orders far from target.

### 3. Dynamic Key Levels (Liquidity footprints)
Identifies dynamic Support and Resistance zones identified through volume-weighted price distribution:
- **Calculation**: Aggregates volume at every price point traded in the session to find the top 5 liquidity centers.
- **Support**: High-volume price levels *at or below* the current close.
- **Resistance**: High-volume price levels *above* the current close.

---
