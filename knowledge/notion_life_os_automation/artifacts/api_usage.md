# Notion API Implementation

## Shared Utilities
Most scripts utilize `httpx` for making requests to the Notion API (`https://api.notion.com/v1`).

## Headers
Required headers for authentication and versioning:
```python
headers = {
    "Authorization": f"Bearer {config.NOTION_TOKEN}",
    "Notion-Version": "2022-06-28",
    "Content-Type": "application/json"
}
```

## Common Operations

### Creating a Page (Logging Data)
Used extensively in `workout_log.py` and `push_water_today.py`.
- **Endpoint**: `POST /v1/pages`
- **Payload**: Requires `parent` (database_id) and `properties`.

### Querying a Database
Used in `list_workouts` and `morning_routines.py`.
- **Endpoint**: `POST /v1/databases/{database_id}/query`
- **Payload**: Can include `filter`, `sorts`, and `page_size`.

### Updating a Page (Checking items)
Used in `morning_routines.py`.
- **Endpoint**: `PATCH /v1/pages/{page_id}`
- **Payload**: Contains the updated `properties` (e.g., checkbox state).

## Search and Update Pattern
Useful for data correction (e.g., fixing a wrong date).

1. **Query**: Find the page ID based on unique properties (Name, Date).
2. **Patch**: Update the specific property on the found page.

Example implementation:
```python
# 1. Search
query_payload = {
    "filter": {
        "and": [
            {"property": "Date", "date": {"equals": "2026-01-29"}},
            {"property": "Name", "title": {"equals": "Indoor Run"}}
        ]
    }
}
r = httpx.post(f"{BASE_URL}/databases/{DB_ID}/query", headers=headers, json=query_payload)
page_id = r.json()["results"][0]["id"]

# 2. Update
update_payload = {"properties": {"Date": {"date": {"start": "2026-01-28"}}}}
httpx.patch(f"{BASE_URL}/pages/{page_id}", headers=headers, json=update_payload)
```

## Data Export for AI Analysis
To facilitate analysis by external LLMs, the system uses a pattern of querying multiple databases and aggregating the results into a single JSON file (`analysis_data.json`).

### Export Workflow
1. **Query**: Fetch relevant entries from targeted databases (e.g., Workout Log, Sleep Health Log) filtered by date or name.
2. **Aggregate**: Combine results into a structured dictionary.
3. **Dump**: Write the dictionary to a `.json` file with `ensure_ascii=False`.

Example:
```python
export_data = {
    "workout_data": workout_query_results,
    "sleep_data": sleep_query_results,
    "timestamp": datetime.now().isoformat()
}
with open("analysis_data.json", "w", encoding="utf-8") as f:
    json.dump(export_data, f, ensure_ascii=False, indent=2)
```
