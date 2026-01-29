# Morning Routines Tracking

## Overview
Morning routines are managed in the "Morning Routines" database in Notion. The script `morning_routines.py` is used to check and monitor these daily tasks.

## Routines List
Identified routines typically include:
1. Dậy 5h (Wake up at 5am)
2. Uống nước (Drink water)
3. Giãn cơ (Stretching)
4. Thiền (Meditation)
5. Viết lịch (Journaling/Planning)
6. Ăn sáng (Breakfast)
7. Omega 3 (Supplement)
8. Đưa con (Drive kids)
9. Cafe (Coffee)

## Script Usage
```bash
python morning_routines.py              # Xem status hôm nay
python morning_routines.py check "Thiền" # Check specific routine
python morning_routines.py all          # Check tất cả routines
```

## Implementation Details
The script interacts with checkboxes in Notion. It can be extended to reset checkboxes weekly or daily using `update_routines.py`.
