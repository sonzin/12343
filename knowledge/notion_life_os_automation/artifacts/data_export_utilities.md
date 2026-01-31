# Data Export Utilities

## Overview
The system provides dedicated utilities for exporting Notion database content into portable formats (CSV and JSON). These are primarily located in the `export/` directory.

## CSV Export Scripts
These scripts use `httpx` to query databases and the built-in `csv` module to save data locally.

### Available Scripts
- **`export/export_all.py`**: Executes export for multiple databases (currently Sleep and Workout).
- **`export/export_sleep.py`**: Specific exporter for the Sleep Health Log.
- **`export/export_workout.py`**: Specific exporter for the Workout Log.

### Usage
```bash
python export/export_all.py
```
Outputs are typically saved to the root directory or a designated `data/` folder as `.csv` files (e.g., `sleep_health_data.csv`, `workout_data.csv`).

## JSON Export (AI-Ready)
For analysis by Large Language Models (LLMs), data is exported in raw JSON format to preserve the full structure of Notion properties.

### Key Pattern
As demonstrated in ad-hoc scripts like `push_and_export.py`, the JSON export pattern involves:
1.  **Direct API Query**: Fetching results directly from the Notion database query endpoint.
2.  **Aggregation**: Grouping results from multiple databases (e.g., matching a workout with the subsequent night's sleep).
3.  **UTF-8 Serialization**: Saving with `ensure_ascii=False` to handle Vietnamese characters.

### Dedicated AI Export Script: `export_fitness_json.py`
To streamline the process, a formal script provides full JSON exports for specific dates:

**Usage:**
```bash
python export_fitness_json.py --workout-date YYYY-MM-DD --sleep-date YYYY-MM-DD
```

**Features:**
- **Full Schema**: Pulls complete Notion page objects, preserving all metadata for deep AI analysis.
- **Flexible Filtering**: Allows independent date selection for workout and sleep data (useful for comparing different days).
- **Automated Directory Handling**: Ensures outputs are saved in the `data/` subdirectory.

### Ad-hoc Aggregate Pattern
In cases where a custom grouping is needed, the following pattern is used:
```python
# Aggregating data for analysis
export_data = {
    "workout": workout_results,
    "sleep": sleep_results,
    "context": "Post-workout recovery analysis"
}
with open("analysis_data.json", "w", encoding="utf-8") as f:
    json.dump(export_data, f, ensure_ascii=False, indent=2)
```
