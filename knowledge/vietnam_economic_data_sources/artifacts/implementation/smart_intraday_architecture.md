# Smart Intraday Monitoring: AI-Ready Architecture

This document describes the architectural design and data structures used in the `smart_intraday.py` system, optimized for AI-driven market analysis and risk management.

## 🕒 Session-Based Aggregation

To provide AI models with meaningful context rather than raw tick data, trading activity is aggregated into logical session blocks.

| Session ID | Time Range | Significance |
|------------|------------|--------------|
| **ATO** | 09:00 - 09:15 | Opening price discovery and overnight sentiment. |
| **MORNING_1** | 09:15 - 10:00 | Early morning volatility and initial trend formation. |
| **MORNING_2** | 10:00 - 10:45 | Mid-morning consolidation or trend continuation. |
| **MORNING_3** | 10:45 - 11:30 | Pre-lunch adjustments and institutional positioning. |
| **AFTERNOON_1** | 13:00 - 14:00 | Mid-day session, often characterized by lower liquidity or "traps." |
| **AFTERNOON_2** | 14:00 - 14:30 | The "Golden Hour" where professional intent becomes clearest. |
| **ATC** | 14:30 - 15:00 | Closing auction; confirms the final professional bias. |

## 📊 AI-Ready JSON Schema

The `smart_intraday.py` script exports a structured JSON format designed for consumption by LLMs (e.g., ChatGPT).

### 1. `day_summary`
Provides a high-level snapshot of the stock's performance.
- `close_price`: Latest matched price.
- `day_high` / `day_low`: Extremes of the session.
- `total_volume`: Total shares traded.
- `vol_vs_20d`: Ratio of current volume vs. 20-day average (e.g., 1.5 = 150%).
- `day_type`: Classification of the day's behavioral pattern (see Enums).

### 2. `key_levels`
Identifies high-liquidity price zones where significant volume was traded.
- `resistance`: Major high-volume levels above current price.
- `support`: Major high-volume levels below current price.

### 3. `trader_context`
Provides actionable insights based on the user's specific portfolio status.
- `position`: `ACTIVE`, `PENDING`, or `NO_POSITION`.
- `risk_mode`: Current risk assessment (see Enums).
- `action_bias`: Recommended bias for decision making (see Enums).

## 🏷️ Standardized Enums

### Day Type (`day_type`)
| Value | Description |
|-------|-------------|
| `ACCUMULATION` | Price stable or rising on high volume; heavy bidding support. |
| `DISTRIBUTION` | Price falling on high volume; heavy active selling. |
| `NEUTRAL` | Low volume or sideways movement without clear intent. |
| `DISTRIBUTION_TO_ACCUMULATION` | Early selling followed by strong absorption/recovery. |
| `ACCUMULATION_TO_DISTRIBUTION` | Early strength followed by high-volume breakdown. |

### Risk Mode (`risk_mode`)
| Value | Context |
|-------|---------|
| `SAFE` | Position is in profit (>3%) with healthy market behavior. |
| `WATCH` | Price is near target or showing early signs of trend change. |
| `WARNING` | Position is in slight loss or showing signs of distribution. |
| `DANGER` | Stop-loss threshold reached or major structural breakdown. |
| `OPPORTUNITY` | Pending stock reaching target price with low risk. |

### Action Bias (`action_bias`)
| Value | Description |
|-------|-------------|
| `HOLD` | Maintain current position. |
| `RIDE` | Trend is strong; stay in for further gains. |
| `CONSIDER_TAKE_PROFIT` | Price near resistance or showing distribution signs. |
| `REVIEW_STOPLOSS` | Price near or below stop-loss; prepare for exit. |
| `READY_TO_BUY` | Pending price reached; check market context. |
| `BUY_SIGNAL` | Pending price reached + Accumulation day type confirmed. |
| `WAIT_FOR_DIP` | Price is way above target buy price. |
| `WAIT_CONFIRM` | Price at target but distribution pressure is high. |

## 🧠 Risk Assessment Logic

The system implements a matrix to determine `risk_mode` and `action_bias`:

### For ACTIVE Positions:
- **Profit > 3%**: Mode: `SAFE`. Bias: `RIDE` (if trend up) or `CONSIDER_TAKE_PROFIT` (if Distribution).
- **-3% < PnL < 0%**: Mode: `WARNING` (if Distribution) else `NEUTRAL`. Bias: `HOLD`.
- **PnL < -5%**: Mode: `DANGER`. Bias: `REVIEW_STOPLOSS`.

### For PENDING Positions:
- **Price > Target * 1.02**: Bias: `WAIT_FOR_DIP`.
- **Target < Price <= Target * 1.02**: Bias: `WATCH`.
- **Price <= Target**: 
    - If DayType = `DISTRIBUTION`: Bias: `WAIT_CONFIRM` (Avoid falling knives).
    - If DayType = `ACCUMULATION`: Bias: `BUY_SIGNAL`.
    - Else: Bias: `READY_TO_BUY`.
