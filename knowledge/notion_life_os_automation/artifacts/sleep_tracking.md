# Sleep Health Tracking

## Overview
Sleep biometric data is logged to the "Sleep Health Log" database in Notion. The primary script for this is `trackers/sleep_log.py`.

## Key Features
- **Biometric Monitoring**: Tracks Heart Rate Variability (HRV), Resting Heart Rate (RHR), SpO2, and Respiratory Rate.
- **Sleep Architecture**: Logs total sleep time, time in bed, and durations for Deep, Light, and REM stages.
- **Quality Metrics**: Captures "Sleep Score", "Deep Sleep Continuity", and "Breathing Quality".
- **Environment Log**: Supports tracking of sleep aids like "Nút tai 3M" (Earplugs) and "White Noise".

## Core Logging Function
```python
from trackers.sleep_log import log_sleep

log_sleep(
    name="Sleep Report",
    sleep_date="2026-01-29",
    score=84,
    rhr=67,
    hrv=46,
    total_sleep_h=7.23,
    night_sleep_h=7.23,
    deep_sleep_h=1.77,
    light_sleep_h=4.2,
    rem_sleep_h=1.27,
    deep_continuity=65,
    breathing_quality=98,
    spo2=97,
    resp_rate=13,
    bed_time="22:27",
    wake_time="05:41",
    notes="Deep sleep fragmented."
)
```

## Database Schema (Selected Properties)
- `Name`: Title of the log.
- `Date`: ISO format date.
- `Sleep Score`: Number.
- `Total Sleep (h)`: Number.
- `RHR (bpm)`: Number.
- `HRV (ms)`: Number.
- `Deep Sleep Continuity`: Number.
- `Nút tai 3M`: Checkbox.
- `White Noise`: Checkbox.

## Analysis and Pattern Insights
The logging system allows for cross-metric analysis over time.

### Exercise and Deep Sleep Correlation
Observations from data logged in late January 2026 suggest a link between physical activity and sleep architecture:
- **Correlation**: Shorter exercise duration on a given day has been observed to correlate with a lower deep sleep ratio that night.
- **Metric Sensitivity**: "Deep Sleep Continuity" and "Deep Sleep Ratio" are key metrics for identifying recovery quality following different intensities of training.

## AI Analytics & Data Constraints
The automated `fitness_ai_report.json` (v3.1) processes sleep logs into qualitative descriptors.

### Sleep Score Factors
Numerical data is mapped to descriptors like `duration: GOOD` or `deep_sleep: LOW` to help AI "debug" recovery issues.

### Known Constraints ("Blind Spots")
- **REM Sleep & Bedtime**: Currently `null` in exports because the Huawei Band 10 does not reliably capture REM, and the Notion schema does not yet include a dedicated `bedtime` field.
- **Biometric Sensitivity**: HRV and RHR are the most reliable indicators for the "Daily Readiness Score", while sleep duration is treated as a secondary constraint.
