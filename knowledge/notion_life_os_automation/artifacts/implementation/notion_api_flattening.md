# Notion API Data Flattening Patterns

When exporting data from Notion for AI analysis, the raw JSON response from the Notion API is often too verbose and complex. This artifact documents the generalized pattern for flattening Notion properties into simple, readable key-value pairs.

## Extracting Property Values

The following mapping shows how to extract actual values from different Notion property types:

| Property Type | Extraction Logic |
| :--- | :--- |
| `title` | `prop["title"][0]["plain_text"]` |
| `rich_text` | `prop["rich_text"][0]["plain_text"]` |
| `number` | `prop["number"]` |
| `select` | `prop["select"]["name"]` |
| `date` | `prop["date"]["start"]` |
| `checkbox` | `prop["checkbox"]` |
| `multi_select` | `[item["name"] for item in prop["multi_select"]]` |

## Python Implementation Pattern

A reusable utility function can simplify the extraction process:

```python
def extract_property_value(prop):
    """Extract the actual value from a Notion property object."""
    if not prop:
        return None
        
    prop_type = prop.get("type")
    
    if prop_type == "title":
        titles = prop.get("title", [])
        return titles[0]["plain_text"] if titles else None
    elif prop_type == "rich_text":
        texts = prop.get("rich_text", [])
        return texts[0]["plain_text"] if texts else None
    elif prop_type == "number":
        return prop.get("number")
    elif prop_type == "select":
        select = prop.get("select")
        return select["name"] if select else None
    elif prop_type == "date":
        date_obj = prop.get("date")
        return date_obj["start"] if date_obj else None
    elif prop_type == "checkbox":
        return prop.get("checkbox")
    elif prop_type == "multi_select":
        items = prop.get("multi_select", [])
        return [item["name"] for item in items]
    else:
        return None
```

## AI-Friendly JSON Structure

Beyond flattening, adding a metadata and summary header helps LLMs parse the data more effectively. This is crucial for large datasets where an AI might otherwise fail to locate specific records.

### Recommended Schema: `fitness_ai_report.json` (v3.1)
The v3.1 schema provides **Baseline-Aware Analytics**, comparing current trends against 30-day personal benchmarks to provide qualitative status and specific training recommendations.

```json
{
  "_metadata": {
    "generated_at": "2026-01-31",
    "version": "3.1",
    "format": "CLEAN - optimized for AI coaching analysis",
    "schema_notes": {
       "hrv_ms": "Heart Rate Variability. Higher is better.",
       "baseline_30d": "30-day average used as personal reference point.",
       "delta_pct": "Percentage change from 30-day baseline.",
       "training_load_7d": "Calculated metric: sum(duration * intensity_factor)"
    }
  },
  "_summary": {
    "total_completed_workouts": 15,
    "trends_7d": {
      "period": "last_7_days",
      "daily_readiness": {
        "score": 82,
        "status": "READY",
        "recommendation": "Good to proceed with planned aerobic or moderate training",
        "factors": { "hrv": "GOOD", "sleep": "EXCELLENT", "load": "BALANCED" }
      },
      "hrv": { 
        "avg_ms": 40.3, 
        "baseline_30d": 39.5,
        "delta_pct": 2.0,
        "status": "NORMAL"
      },
      "resting_heart_rate": {
        "avg_bpm": 66.7,
        "baseline_30d": 66.5,
        "delta_pct": 0.3,
        "trend": "STABLE"
      },
      "training_load_7d": { 
        "value": 159.3, 
        "status": "OPTIMAL",
        "recommendation": "Maintain current training volume..."
      }
    }
  },
  "workouts": [
    { 
      "date": "2026-01-31", 
      "hr_zones": { "aerobic_min": 15, "fat_burning_min": 25 }
    }
  ],
  "sleep_logs": [
    {
      "date": "2026-01-31",
      "sleep_score_factors": { "deep_sleep": "GOOD", "duration": "LOW" }
    }
  ]
}
```

This structure allows the AI to immediately grasp the context, time scale, and **readiness state** of the user before processing individual records.

## Implemented Advanced Metrics (v3.0)

### 1. Daily Readiness Score
Reads daily health signals to produce a single actionable recommendation:
- **Scoring Method**: Base 70 + HRV(±15) + RHR_trend(±5) + Sleep(±10) + Load_fatigue(±10).
- **Statuses**:
  - `READY_TO_PUSH` (>= 85): Prime for high intensity.
  - `READY` (70-84): Proceed with moderate training.
  - `MODERATE` (55-69): Light training or recovery walk.
  - `RECOVERY_NEEDED` (< 55): Rest day recommended.

### 2. Training Load & Readiness Mapping
Training load provides a proxy for physical stress, now categorized by volume:
- **Method**: `sum(duration_minutes * intensity_factor)`
- **Intensity Mapping**:
  - `avg_hr < 110`: factor 0.5 (Recovery/Warmup)
  - `avg_hr 110-130`: factor 1.0 (Aerobic/Base)
  - `avg_hr > 130`: factor 1.5 (Anaerobic/High Effort)
- **Status Thresholds**:
  - `< 80`: `LOW` (Recommendation: Add light aerobic)
  - `80 - 200`: `OPTIMAL` (Recommendation: Maintain volume)
  - `> 200`: `HIGH` (Recommendation: Increase recovery)

### 3. Baseline-Aware HRV/RHR
Comparing current 7-day averages against 30-day baselines allows for precise recovery detection:
- **HRV Status**:
  - `ABOVE_BASELINE` (Delta > +5%): Excellent recovery.
  - `NORMAL` (Delta -10% to +5%): Stable state.
  - `BELOW_BASELINE` (Delta < -10%): Compromised recovery.
- **RHR Trend**: Spikes > 5bpm from 30-day baseline trigger fatigue/stress warnings.

### 4. HR Zone Parsing
Zone data is extracted from unstructured "Notes" using regex:
- `Aerobic: (\d+)m` -> `aerobic_min`
- `Fat-burning: (\d+)m` -> `fat_burning_min`

### 5. Sleep Factor Analysis
Converts numerical scores into qualitative descriptors:
- **Deep Sleep Ratio**: `deep_hours / (deep_hours + light_hours)`. Target > 20%.
- **Duration**: `total_hours >= 7` (GOOD), `6-7` (NORMAL), `< 6` (LOW).

## Handling Source Data Constraints ("Blind Spots")
AI models consuming this data should be aware of known limitations in the source (Notion/Hardware):
1. **Missing Metrics**: Fields like `rem_sleep_hours` or `bedtime` may be `null` if the tracking hardware (e.g., Huawei Band 10) or Notion schema does not capture them.
2. **Ambiguous Notes**: While regex extracts HR zones, non-standard note formats may fail to parse. The `avg_heart_rate` remains the fallback for load calculation.
3. **Hardware Inconsistency**: Data attributed to different devices (e.g., Mi Band 4 vs. Huawei B10) may have different accuracy levels.

### Troubleshooting AI "Missing Data" Hallucinations
Even with clean JSON, AI models may sometimes claim specific dates are missing. To mitigate this:
1.  **Explict `all_dates` List**: The `_summary` should contain an exhaustive list of all dates present in each category.
2.  **Summary-First Prompting**: Instruct the AI to check the `_summary` section *before* parsing individual records.
3.  **Unified Date Format**: Ensure all dates follow the `YYYY-MM-DD` ISO format strictly to avoid parsing ambiguity.
4.  **Verification Steps**: If an AI claims data is missing, ask it to look at the `_summary.workout_date_range.all_dates` array explicitly.
