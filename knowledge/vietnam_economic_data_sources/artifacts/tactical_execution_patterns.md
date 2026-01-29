# Tactical Execution Patterns for Vietnam Stocks

This document captures tactical trade execution and order management patterns identified through AI-driven portfolio analysis and market observation in the 2025-2026 cycle.

## 📉 Order Management Patterns

### 1. Pullback-Oriented Bidding (MA20 Logic)
When placing "Pending" buy orders for stocks exhibiting upward momentum but not yet in an overbought state:
- **Rule**: Avoid "chasing" the current price if it is significantly above the **MA20** (20-day Moving Average).
- **Tactical Adjustment**: Set the bid price at or slightly above the MA20 level (e.g., 75,000-75,500 if MA20 is at 75,150) to increase the probability of a "clean" entry during a standard market pullback.
- **Rationale**: Reduces the risk of "buying the top" of a short-term rally and improves the risk/reward ratio as the MA20 often acts as dynamic support in a healthy uptrend.

### 2. Stop-Loss Discipline (The 5% Threshold)
- **Standard Trigger**: Evaluate a strict exit if the P/L drops to **-5%**.
- **Structural Trigger**: Monitor key technical levels (e.g., support at 23,000 for a 25,000 entry). If price breaks support with high volume (>150% 20-day average), favor an immediate exit over "hoping" for a bounce.
- **RSI Context**: A stop-loss break accompanied by an RSI falling below 40 indicates weakening momentum and higher probability of further downside.

### 3. Real-Time VPS Alerting (The "Silent Watcher" Pattern)
For efficient risk management, automated scripts on VPS should follow a high-frequency, low-noise pattern:
- **Scan Frequency**: 1-minute intervals (High-frequency) to capture rapid price movements.
- **Trading Window Constraint**: Only execute during active market hours (09:00-11:30 and 13:00-15:00) to conserve resources.
- **"Silent" Logic**: The system remains silent unless a specific threshold is crossed:
  - **Stop-Loss/Take-Profit**: Triggered when P/L reaches configured thresholds (e.g., -5% or +10%).
  - **Pending Match**: Triggered when the current market price falls within a **0.5% buffer** of a "Pending" order's entry price (`current_price <= target_price * 1.005`). This provides lead time for manual order adjustment or execution.
- **State Management (Anti-Spam Logic)**: To prevent notification fatigue, the monitor implements a persistent state in `data/alert_state.json`.
  - **Key Structure**: `symbol_alerttype_date` (e.g., `ACB_STOP_LOSS_2026-01-28`).
  - **Logic**: The script only dispatches a Telegram message if this unique key is absent from the state file, resetting the state daily.

### 4. Warning & Early Indicators
- **Warning Loss**: Triggered at **-3%** to provide situational awareness before the hard stop-loss at -5%. Usually sent as a silent notification.
- **Take Profit (Standard)**: Triggered at **+10%** for mid-term swing trades. For high-growth stocks like FPT, this may be adjusted to +15% or +20% based on Strategy 2026 context.

## 📊 Portfolio Composition Strategy

### 1. The "Tripod" Model (Kiềng 3 Chân)
To avoid high concentration risks (e.g., 100% Banking):
- **Core (Blue-chip)**: 40% (e.g., VCB, ACB, BID) - Sensitivity to macro/indices.
- **Swing (Mid-cap/Cyclical)**: 35% (e.g., HPG, KBC, PVS) - Higher volatility, sector-specific catalysts.
- **Growth/Innovation**: 25% (e.g., FPT, MWG) - Long-term structural trends.

### 2. Diversification Targets (By Sector)
In the FTSE Upgrade cycle (2025-2026), a balanced portfolio should ideally target:
- **Banking**: 25-30%
- **Materials/Infrastructure**: 15-20%
- **IT/Digital**: 15-20%
- **Industrial RE**: 10-15%
- **Energy/Oil & Gas**: 10-15%

## ⏱️ Intraday Monitoring Discipline

Effective intraday monitoring focuses on high-value data to avoid "data spam" and facilitate clear decision-making.

### 1. Essential Data Subset (Tick-by-Tick)
To accurately read market behavior (tape reading), focus on:
- **Timestamp**: Precision to minutes.
- **Matched Price & Volume**: To see the specific interaction at price levels.
- **Side (Buy/Sell/Unknown)**: Crucial for determining if volume is aggressive buying or forced selling.
- **Accumulated Metrics**: Total volume and value to assess relative significance.

### 2. Frequency Strategy: "Aggregation over Real-time"
Continuous tracking is counter-productive for portfolio monitoring.
- **15-30 Minute Intervals**: Sufficient for identifying traps and absorption/distribution patterns.
- **Event-Based Triggers**: Aggressive monitoring should only activate when:
    - Price touches critical technical support/resistance levels.
    - Volume exhibits an intraday spike (relative to previous candles).
    - Closing Window (ATC): The 14:30-14:45 window is critical for identifying "closing intent."

### 3. AI as the "Risk Manager"
The role of AI in intraday monitoring is not automated trading, but **Dynamic Risk Assessment**:
- **Identify Danger/Safety**: Determining if a price drop is "low-volume drift" or "high-volume dump."
- **Avoid FOMO/Panic**: Providing objective analysis to prevent emotional reactions to short-term volatility.

### 4. Advanced Behavioral Indicators
When reviewing high-resolution intraday data (e.g., via `smart_intraday.py`), focus on these market states:

| State | Indicators | Tactical Significance |
|-------|------------|-----------------------|
| **Opening Behavior** | High SELL volume in first 1-15 mins with failing bids. | Indicates active selling intent; higher probability of a red day regardless of the previous close. |
| **Shakeout (Rũ hàng)** | Price drops on very low volume, often followed by absorption at support. | Not a reason to exit; often a precursor to a "Bear Trap." |
| **Panic Sell** | Rapid price drop across 10-15 mins with exponential volume growth. | High risk of a sustained downtrend; secondary support levels likely to fail. |
| **Controlled Distribution** | Steady, large-volume blocks sold over mid-session (10:00-14:00). | Warning sign; institutional exiting without crashing the price immediately. |
| **Closing Intent (ATC)** | Directional move in last 15 mins (14:45-15:00) with volume support. | Confirms the professional bias for the next session. "The Amateur opens the market, the Professional closes it." |

### 5. The "Optimal Monitor Combo" (Dành cho Trader Cá Nhân)
To achieve high-resolution analysis without information overload, the following data points should be combined:
1. **Intraday Tick/Smart Blocks**: To see session behavior (5-15 min blocks).
2. **VNINDEX Context**: Brief snapshot of the overall market (Price, Vol, Width). Stocks dying in a weak market are "normal"; stocks dying in a strong market are "danger."
3. **Volume vs. 20-Day Average**:
    - `Price Down + Vol < 70% Avg`: Likely a **Shakeout**.
    - `Price Down + Vol > 150% Avg`: Likely **Distribution/Panic**.
4. **Institutional Context**: Tracking Net Foreign and Net Proprietary trading for the session. Professional accumulation often hides behind mid-session distribution of retail players.
5. **ATC Specifics**: Monitor the % of daily volume traded in the closing auction to judge "Institutional Defensiveness."
6. **Personal Context**: Current holding status, margin levels, and trade objectives (Swing vs. T+).

### 7. Professional Flow Interpretation Matrix
When combining the data layers from the "Standard Feed," the analyst (or AI) should identify these specific institutional intent patterns:

| Pattern | Correlation | Professional Intent | Tactical Action |
|---------|-------------|---------------------|-----------------|
| **Panic Absorption** | Price Down + VNINDEX Down + Net Foreign/Prop Buy. | Institutions are defending the stock despite a broad market crash. | **HOLD/BUY**: High probability of "Outperforming" when the market stabilizes. |
| **Passive Distribution** | Price Stable + VNINDEX Up + High Volume + Net Institutional Sell. | Professionals are liquidating positions into "dumb money" retail strength. | **REDUCE/TIGHTEN STOP**: The stock is losing institutional support despite green price. |
| **The "Blind Dump"** | Price Down + VNINDEX Up + Net Institutional Sell. | Direct liquidation; institutions want out at any cost. | **EXIT IMMEDIATE**: High danger signal; fundamental change likely. |
| **Institutional Sprint** | Price Up + VNINDEX Up + Net Foreign Buy + Big Order support. | Coordinated professional accumulation during a bullish trend. | **RIDE**: Strong trend; trailing stop-loss recommended. |

- **Rule of Thumb**: The most reliable signals occur when **Institutional Flow** (Layer 3) contradicts the **Broad Market** (Layer 2) in a way that protects the price of your specific stock.

### 8. Automated Risk Bias & Action Logic
When analyzing a position, apply these system-automated triggers based on the **"Smart Feed"**:

| Position State | Condition | Risk Mode | Action Bias |
|----------------|-----------|-----------|-------------|
| **Active**     | P/L > +3% (Sprint/Accum) | **SAFE** | HOLD / RIDE |
| **Active**     | P/L > +3% (Distribution) | **NEUTRAL** | CONSIDER TAKE PROFIT |
| **Active**     | -3% < P/L < 0% (Neutral) | **NEUTRAL** | HOLD |
| **Active**     | -3% < P/L < 0% (Dist.) | **WARNING** | MONITOR CLOSELY |
| **Active**     | -5% < P/L < -3% | **WARNING** | MONITOR CLOSELY (Alert) |
| **Active**     | P/L < -5% | **DANGER** | REVIEW STOPLOSS (Exit) |
| **Active**     | P/L < 0% (Dist. to Accum) | **NEUTRAL** | WAIT CONFIRM (Don't panic) |
| **Pending**    | Price <= Target | **OPPORTUNITY** | READY_TO_BUY |
| **Pending**    | Price <= Target * 1.02 | **WATCH** | APPROACHING_TARGET |
| **Pending**    | Price > Target * 1.02 | **NEUTRAL** | WAIT_FOR_DIP |
| **Pending**    | Distribution (Any Price) | **WATCH** | **WAIT_CONFIRM** (Breakdown Risk) |
| **Pending**    | Accumulation + Price <= Target | **OPPORTUNITY** | **BUY_SIGNAL** |

- **Falling Knife Prevention**: Even if a stock hits its target entry price (e.g., CTD at 76,000), if the automated **Day Type** is classified as `DISTRIBUTION` (active selling pressure), the system triggers a **WAIT_CONFIRM** override. This prevents entering a position right before a potential structural breakdown.

- **State Reset**: These biases should be evaluated at the end of each session block (e.g., after MORNING_3 or AFTERNOON_2) to adjust the stop-loss order's proximity.

### 9. Macro-Technical Decoupling (Vĩ mô vs. Kỹ thuật)
A critical insight for the 2026 cycle is the frequent "decoupling" between strong structural macro fundamentals and irrational technical sell-offs:
- **The Pattern**: Market crashes (e.g. -27 points on 28/01/2026) driven by localized negative news (Vingroup shocks) despite 8% GDP growth and record low inflation (3.3%).
- **Tactical Rule**: When technical supports fail (-5% stop-loss triggered) but macro remains intact, separate the **Trading Portfolio** (Exit immediately) from the **Investment Portfolio** (Accumulate slowly). 
- **The "Bệ đỡ" (Pillar)**: Strong macro serves as the ultimate support; technical sell-offs without macro deterioration are high-probability recovery setups (The "U-shape" recovery).

### 10. The Liquidity Paradox (Nghịch lý Thanh khoản)
In high-resolution analysis, an increase in liquidity during a severe price drop is not always bearish:
- **Scenario A (Bearish)**: High Vol + Price Drop + Low institutional buying = **Liquidation/Panic**.
- **Scenario B (Bullish Absorption)**: High Vol + Price Drop + High institutional net buying (Foreign/Prop buy) = **Panic Absorption**. 
- **Observation**: 28/01/2026 showed a "Liquidity Paradox" where VN-Index plummeted but Tự doanh (Proprietary) bought ròng +72B in VIC at its floor, indicating professional absorption of retail panic.
