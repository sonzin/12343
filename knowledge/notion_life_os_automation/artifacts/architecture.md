# Notion Life OS: Architecture & Configuration

## Project Location
Local directory: `/Users/sonho/Library/CloudStorage/GoogleDrive-quangson.censtaf@gmail.com/Drive của tôi/Code/Personal Notion/NotionLifeOS`

## Configuration (`config.py`)
The system centralizes all configuration in `config.py`. Key components include:

- **Notion Token**: `NOTION_TOKEN` (used for all API requests).
- **Database IDs**: Mapped to human-readable keys for easy script access.
  - `workout_log`: `2ed8e7e5-6102-81dc-b50c-df5579310f0d`
  - `water_tracking`: `0866b57e69fa429fadf97e3691f1a1ab`
  - `morning_routines`: `93675e0c-dd0e-4523-9718-969a41bf5b4e`
  - `daily_log`: `ef11f9ab92744c378f8fa7a2d092d5f7`
  - `books`: `4b1535ea-11cb-43c9-b06e-0dfcbc0b7696` (Primary)
- **Targets**: Health and productivity goals like steps (7500), water (1500ml), and exercise (30 min).

## Folder Structure
- `core/`: Base classes and shared utilities.
- `trackers/`: Specific tracking scripts (e.g., `workout_log.py`, `morning_routines.py`, `sleep_log.py`).
- `export/`: Scripts for data exportation to CSV (general) and JSON (AI-ready).
- `data/`: Local storage for data before syncing.
- `scripts/`: Maintenance and utility scripts.
- `.agent/workflows/`: Documentation for AI agents.
