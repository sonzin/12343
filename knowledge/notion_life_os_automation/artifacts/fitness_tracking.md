# Fitness Tracking: Workout Log

## Overview
Fitness data is pushed to the "🏃 Workout Log" database in Notion. The primary script for this is `trackers/workout_log.py`.

## Key Features
- **Comprehensive Metrics**: Supports distance, duration, calories, heart rate (AVG/MAX), pace, cadence, and HR zones.
- **Device Attribution**: Tracks which device recorded the data (default: "Huawei B10").
- **Multiple Views**: Includes a `list_workouts` function to display recent data in the terminal.

## Core Logging Function
```python
from workout_log import log_workout

log_workout(
    name="Afternoon Run",
    workout_date="2026-01-29",
    bai_tap="Cardio/Running",
    duration_min=45,
    distance_km=5.2,
    hr_avg=145,
    hr_max=160,
    notes="Felt strong, steady pace."
)
```

## Database Schema (Selected Properties)
- `Name`: Title of the workout.
- `Date`: ISO format date.
- `Status`: Select (default "Đã tập").
- `Cự ly (km)`: Number.
- `HR TB (bpm)`: Number.
- `Zone Fat Burning`: Number (minutes in zone).
- `Thiết bị`: Select (e.g., "Huawei B10").

## Database Schema Limitations & Troubleshooting
During data synchronization (e.g., Jan 29, 2026), it was discovered that the target Notion database may lack certain properties expected by the `workout_log.py` script. 

### Missing Properties
The following properties were identified as missing in the current Notion setup:
- `Day`
- `Active Calories`
- `Avg Pace (min/km)`
- `Fastest Pace`
- `Avg Speed (km/h)`
- `HR Max (bpm)`
- `Avg Cadence (spm)`
- `Max Cadence`
- `Avg Stride (cm)`
- `Zone Aerobic`
- `Zone Fat Burning`

### Workaround
When pushing data with these metrics, the script should be configured to move unsupported fields into the `Notes` (Ghi chú) field to avoid API errors while preserving the data.

### Date Correction
If a workout is logged on the wrong date, it can be corrected by:
1.  Querying the database for the entry by name and the (incorrect) date.
2.  Using the `id` from the result to send a `PATCH` request to the `/v1/pages/{page_id}` endpoint with the corrected date.

## Automated AI Analytics & Readiness
As of v3.1 of the export system, fitness records are automatically analyzed to produce a **Daily Readiness Score** (0-100).

- **Training Load**: Workouts are weighted by duration and heart rate intensity to calculate a 7-day rolling load.
- **Readiness Mapping**: The score aggregates HRV status, RHR trends, and training load to provide a "HLV nói 1 câu" (Coach's one-liner) recommendation (e.g., `READY_TO_PUSH` or `RECOVERY_NEEDED`).
- **Coach-Ready Export**: The processed results are available in `data/fitness_ai_report.json` for external AI coaching.

## Planned Workouts (Dự kiến)
AI agents can generate and push "Planned" workouts based on the Daily Readiness Score.

### Workflow
1.  Analyze the **Daily Readiness Score** and **7-day Training Load**.
2.  Generate a workout recommendation (e.g., Aerobic Zone 2 if readiness is high).
3.  Push to Notion with **Status**: `Dự kiến`.
4.  Include detailed targets in the `Notes` (Ghi chú) property, such as:
    - **Warm-up**: 5-7 mins (HR < 110)
    - **Main Set**: 25-30 mins (HR 115-125)
    - **Rationale**: References to readiness score (e.g., 90/100) and HRV trends.

### Recovery Decision Logic
When the **Training Load** is `OPTIMAL` and the goal is to maximize recovery (parasympathetic activation), AI agents may pivot from indoor cardio to low-impact activities:
- **Swimming (Bơi lội)**: Preferred for active recovery as it has low joint impact and promotes deep breathing.
- **Goal**: Maintain circulation without adding mechanical stress to the legs.
- **Deep Sleep Correlation**: Longer duration, low-intensity sessions (e.g., 40-45 mins) have shown a positive correlation with increased **Deep Sleep** hours in this specific Life OS user's data.
