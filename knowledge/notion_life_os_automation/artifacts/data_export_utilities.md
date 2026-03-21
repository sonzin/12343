# Data Export Utilities

## Overview
The system provides dedicated utilities for exporting Notion database content into portable formats (CSV and JSON). These are primarily located in the project root and `data/` directory.

## Primary AI Export Script: `export_fitness_clean.py`

**Location:** `/NotionLifeOS/export_fitness_clean.py`
**Output:** `/NotionLifeOS/data/fitness_ai_report.json`

### Usage
```bash
cd NotionLifeOS
python3 export_fitness_clean.py
```

### Current Version: v3.1 (2026-01-31)

**Features:**
1. **Flattened Data**: Converts verbose Notion API response to simple key-value objects
2. **HR Zones Parsing**: Auto-extracts zone info from notes (aerobic_min, fat_burning_min, etc.)
3. **Sleep Score Factors**: Adds breakdown (deep_sleep: GOOD/NORMAL/LOW, duration: GOOD/NORMAL/LOW)
4. **7-Day Trends**: Rolling averages for HRV, RHR, training load, sleep
5. **30-Day Baseline**: Personal baseline for HRV/RHR with delta percentage
6. **Daily Readiness Score**: 0-100 score with factors and recommendation

### Output Structure
```json
{
  "_metadata": { "version": "3.1", "schema_notes": {...} },
  "_summary": {
    "total_workout_entries": 18,
    "total_completed_workouts": 15,
    "workout_date_range": { "earliest": "...", "latest": "...", "all_dates": [...] },
    "sleep_date_range": { "earliest": "...", "latest": "...", "all_dates": [...] },
    "trends_7d": {
      "daily_readiness": { "score": 90, "status": "READY_TO_PUSH", "recommendation": "..." },
      "hrv": { "avg_ms": 40.3, "baseline_30d": 39.5, "delta_pct": 2.0, "status": "NORMAL" },
      "resting_heart_rate": { "avg_bpm": 66.7, "baseline_30d": 66.5, "trend": "STABLE" },
      "training_load_7d": { "value": 159.3, "status": "OPTIMAL", "recommendation": "..." },
      "sleep_7d": { "avg_score": 82.4, "avg_deep_sleep_hours": 1.88 }
    }
  },
  "workouts": [...],
  "sleep_logs": [...]
}
```

### Daily Readiness Scoring Method
```
Base 70 + HRV(±15) + RHR_trend(±5) + Sleep(±10) + Load_fatigue(±10)
```
- **READY_TO_PUSH** (≥85): High-intensity training OK
- **READY** (≥70): Normal training OK  
- **MODERATE** (≥55): Light training recommended
- **RECOVERY_NEEDED** (<55): Rest day

### Data Limitations
The following fields may be `null` due to source data constraints:
- `rem_sleep_hours`: Huawei Band 10 doesn't measure REM
- `bedtime`: Not captured in Notion database
- Early entries from Mi Band 4 may have distance inaccuracies

### Legacy Scripts (Deleted)
- `export_fitness_json.py` - Replaced by export_fitness_clean.py
- `export_recent_data.py` - Replaced by export_fitness_clean.py
- Old CSV exports in `data/` folder - No longer needed

## When to Use
Run `export_fitness_clean.py` when you need to:
- Feed fitness data to ChatGPT/Gemini/Claude for analysis
- Check daily readiness before workout
- Review weekly training trends
