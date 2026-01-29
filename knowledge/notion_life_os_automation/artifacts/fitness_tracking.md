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
