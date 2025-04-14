
### ✅ **Why and When to Use These Extra Types**

| **Data Type**       | **Use Case**                                                                                   | **FastAPI Conversion**                     |
|---------------------|------------------------------------------------------------------------------------------------|--------------------------------------------|
| `UUID`              | Unique resource identifiers (e.g., user ID, product ID). Safe for distributed systems.         | Comes in as `str`, auto-converted to `UUID`|
| `datetime.datetime` | Timestamps for events (e.g., order created at, updated at).                                     | ISO 8601 string → `datetime` object        |
| `datetime.date`     | Just a calendar date, no time (e.g., date of birth).                                           | ISO string → `date`                        |
| `datetime.time`     | Specific times (e.g., repeat reminder time, alarm time).                                       | ISO string → `time`                        |
| `datetime.timedelta`| Time intervals (e.g., delay processing for X minutes).                                         | ISO string or float → `timedelta`          |
| `bytes`             | Raw binary data (e.g., files, images).                                                         | Comes as `str`, interpreted as base64      |
| `Decimal`           | Precise numbers (e.g., money, scientific calculations).                                        | Like float, but avoids rounding issues     |
| `frozenset`         | Unique, immutable sets (e.g., unique tags).                                                    | List in, set conversion + uniqueness       |

---

### ✅ Example Breakdown (Refresher)

Here’s the example code again with inline comments to explain what’s going on:

```python
from datetime import datetime, time, timedelta
from typing import Annotated
from uuid import UUID
from fastapi import Body, FastAPI

app = FastAPI()

@app.put("/items/{item_id}")
async def read_items(
    item_id: UUID,  # Accepts UUID string like "123e4567-e89b-12d3-a456-426614174000"
    start_datetime: Annotated[datetime, Body()],  # Accepts "2025-04-12T15:00:00"
    end_datetime: Annotated[datetime, Body()],
    process_after: Annotated[timedelta, Body()],  # Accepts duration like "PT30M"
    repeat_at: Annotated[time | None, Body()] = None  # Optional, time like "14:23:55"
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
        "item_id": item_id,
        "start_datetime": start_datetime,
        "end_datetime": end_datetime,
        "process_after": process_after,
        "repeat_at": repeat_at,
        "start_process": start_process,
        "duration": duration,
    }
```

---

### 💡 Tips

- Use these types in models too, not just function parameters.
- Use `datetime` for anything time-sensitive — it'll save you from timezone bugs and string manipulation.
- Use `UUID` over `int`/`str` IDs for better security and uniqueness.
- Use `Decimal` for financial values to avoid float rounding issues.

---
