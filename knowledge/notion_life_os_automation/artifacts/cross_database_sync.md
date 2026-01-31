# Cross-Database Data Synchronization

## Overview
As the Notion Life OS matures, it is often necessary to synchronize health metrics across different databases to provide contextual insights. A prime example is syncing sleep metrics (HRV, RHR) into the Workout Log to analyze recovery state against physical performance.

## Synchronization Pattern: Sleep to Workout
This pattern uses the `Date` property as a common key to match sleep reports with workout sessions.

### Workflow
1. **Fetch Sleep Data**: Query the Sleep Health Log for recent entries, building a mapping of `Date -> {HRV, RHR}`.
2. **Fetch Workout Data**: Query the Workout Log for recent entries.
3. **Match and Patch**:
   - For each workout, check if a corresponding sleep report exists for the same date.
   - If a match is found, compare the current workout properties (`HRV (ms)`, `RHR Sang (bpm)`) with the sleep data.
   - If update is needed, send a `PATCH` request to the workout page.

### Implementation Example (`sync_sleep_to_workout.py`)
```python
def sync_data():
    # 1. Map Sleep Date -> Metrics
    sleep_map = {p['date']: {'hrv': p['hrv'], 'rhr': p['rhr']} for p in sleep_results}

    # 2. Iterate through workouts
    for workout in workout_results:
        date_str = workout['date']
        if date_str in sleep_map:
            data = sleep_map[date_str]
            updates = {
                "HRV (ms)": {"number": data['hrv']},
                "RHR Sáng (bpm)": {"number": data['rhr']}
            }
            # Patch the workout page
            httpx.patch(f"{BASE_URL}/pages/{workout['id']}", json={"properties": updates})
```

## Benefits
- **Consolidated Health Context**: Allows users to see recovery data directly alongside workout data without manual entry.
- **Correlative Analysis**: Enables the AI analysis scripts to easily detect correlations (e.g., "A night of low HRV was followed by a lower-intensity run").
- **Schema Parity**: Ensures that important health metrics are available in all relevant trackers.

## Verification and Consistency
To ensure data integrity, especially when matching by `Date`, a verification step is performed:

1. **Full JSON Export**: Use `export_fitness_json.py` to fetch a comprehensive view of both databases for the target date.
2. **Property Inspection**: Compare the `HRV (ms)` and `RHR` values in the workout entry against the corresponding sleep entry.
3. **Example Success**: On 2026-01-30, verification confirmed that a workout log and a sleep log with identical dates were successfully synced (HRV 39, RHR 67), matching the source sleep data.
